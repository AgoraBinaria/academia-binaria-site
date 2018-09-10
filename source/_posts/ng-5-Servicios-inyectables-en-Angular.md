---
title: Servicios inyectables en Angular
permalink: servicios-inyectables-en-Angular
date: 2017-11-23 10:44:58
tags:  
- Angular
- Servicios
- DI
- Tutorial
- Introducción
- Angular6
- Angular5
- Angular2
categories:
- [Tutorial, Angular] 
thumbnail: /css/images/angular-5_inject.png
---

![Tutorial Angular 5-Inject](/images/tutorial-angular-5_inject.png)

La presentación, la lógica y el manejo de datos son tres capas de abstracción que usamos los programadores para mantener organizado nuestro código. En Angular 6, la presentación es cosa de los componentes. **La lógica y los datos tienen su lugar en servicios compartidos**.

Para que los componentes consuman los servicios de forma controlada tenemos _inyectables_ en la librería `@angular/core` con los que realizar **la inyección de dependencias**.

<!-- more -->

Partiendo de la aplicación tal cómo quedó en [Flujo de datos entre componentes Angular](../flujo-de-datos-entre-componentes-angular/). Al finalizar tendrás una aplicación que comunica componentes entre páginas, reparte responsabilidades y gestiona claramente sus dependencias.

> Código asociado a este artículo en _GitHub_: [AcademiaBinaria/AutoBot/5-inject](https://github.com/AcademiaBinaria/autobot/tree/5-inject/)
> > Tienes una versión desplegada operativa para probar [AutoBot](https://academiabinaria.github.io/autobot/) 

# 1. Servicios

Como casi todo en Angular, **los servicios son clases TypeScript**. Su propósito es contener lógica de negocio, clases para acceso a datos o utilidades de infraestructura. Estas clases son perfectamente instanciables desde cualquier otro fichero que las importe. Pero, Angular nos sugiere y facilita que usemos su sistema de inyección de dependencias.

Este sistema se base en convenios y configuraciones que controlan la instancia concreta que será inyectada al objeto dependiente. Ahora verás cómo funciona **[la Dependency Injection](https://es.wikipedia.org/wiki/Inyecci%C3%B3n_de_dependencias) en Angular**.

## 1.1 Inyectables

La particularidad de las clases de servicios está en su decorador: `@Injectable()`. Esta función viene en el `@angular/core` e **indica que esta clase puede ser inyectada** dinámicamente a quien la demande. Aunque es muy sencillo crearlos a mano, el CLI nos ofrece su comando especializado para crear servicios:

```shell
ng g s views/operations/operations
```

El resultado es el fichero `operations.service.ts` que he rellenado con un contenido como este:

```typescript
import { Injectable } from "@angular/core";

@Injectable()
export class OperationsService {
  private operations: Operation[] = [];

  constructor() {}

  public getNumberOfOperations(): number {
    return this.operations.length;
  }
  public getOperationsList(): Operation[] {
    return this.operations;
  }
  public getOperationById(id: string): Operation {
    return this.operations.find(o => o._id === id);
  }
  public saveOperation(operation: Operation) {
    operation._id = new Date().getTime().toString();
    this.operations.push(operation);
  }
  public deleteOperation(operation: Operation) {
    const index = this.operations.indexOf(operation);
    this.operations.splice(index, 1);
  }
}
```

Ahora tienes centralizado en este servicio toda la lógica de datos. Los demás componentes la podrán utilizar e incluso podrán compartir los datos en memoria, como el *array* de operaciones.

## 1.2 Providers

Declarar y decorar la clase no es suficiente. Necesitas **registrarla como un proveedor en algún módulo**. Por ahora hazlo en el cercano módulo de Operaciones usando el array `providers:[]`.

> Ya no es necesario en Angular 6

```typescript
@NgModule({
  imports: [CommonModule, FormsModule, OperationsRoutingModule],
  declarations: [ OperationsComponent, NewComponent, ListComponent, ItemComponent],
  providers: [OperationsService]
})
export class OperationsModule {}
```

A partir de este momento cualquier otro servicio o componente de este módulo que lo reclame **será proveído con una misma instancia** de este servicio. Se crea un [*singleton*](https://es.wikipedia.org/wiki/Singleton) por cada módulo en el que se provea un servicio. Normalmente si el servicio es para un sólo módulo funcional se provee en este y nada más. Algunos servicios de uso común se proveen en el módulo raíz, garantizando así su disponibilidad en cualquier otro módulo de la aplicación.

# 2. Dependencias

Al consumo de los servicios inyectables se le conoce como dependencia. Cada componente o servicio puede **declarar en su constructor sus dependencias** hacia servicios inyectables.

Por ejemplo en el componente `OperationsComponent` teníamos incrustada toda la lógica y mantenimiento de los datos. Debe quedarse solamente con sus responsabilidades de presentación y delegar en el nuevo servicio todo lo demás.

```typescript
export class OperationsComponent implements OnInit {
  public numberOfOperations = 0;
  public operations: Operation[] = [];

  constructor(private operationsService: OperationsService) {}

  ngOnInit() {
    this.refreshData();
  }
  public saveOperation(operation: Operation) {
    this.operationsService.saveOperation(operation);
    this.refreshData();
  }
  public deleteOperation(operation: Operation) {
    this.operationsService.deleteOperation(operation);
    this.refreshData();
  }
  private refreshData() {
    this.numberOfOperations = this.operationsService.getNumberOfOperations();
    this.operations = this.operationsService.getOperationsList();
  }
}
```

Como ves, **el constructor no tiene otra función que la de recibir las dependencias**. Una vez construida la instancia se puede acceder a ellas a través de `this.operationsService`. Ahora este componente ya no sabe nada sobre dónde se almacenan o cómo se recuperan los datos.

## 2.1 Singleton

Lo mismo que le ocurre al `OperationsComponent` le puede pasar a cualquier otro componente del módulo. Como por ejemplo el `ItemComponent`. El cual **reclama la misma dependencia y recibe la misma instancia**. Esto es así porque cada módulo gestiona las dependencias en modo *Singleton*, y entrega a todos los componentes la misma instancia del servicio.

```typescript
export class ItemComponent implements OnInit {
  public operation: Operation;

  constructor(
    private route: ActivatedRoute,
    private operationsService: OperationsService
  ) {}

  ngOnInit() {
    const id = this.route.snapshot.params["id"];
    this.operation = this.operationsService.getOperationById(id);
  }
}
```

> Ojo, si se provee la misma clase en dos o más módulos se genera una instancia en cada uno de ellos. Los componentes recibirán la instancia del módulo jerárquicamente más cercano.

## 2.2 Comunicación via url

Para finalizar el ejercicio te muestro cómo desde el `ListComponent` puedes crear **enlaces que envían parámetros a otras páginas**. Con esa mínima información la página destino puede usar el valor del parámetro para consultar datos en el servicio.

```html
<tbody>
    <tr *ngFor="let operation of operations">
      <td><a [routerLink]="[operation._id]">{{ operation._id }}</a></td>  
      <td>{{ operation.description }}</td>
      <td>{{ operation.kind }}</td>
      <td>{{ operation.amount | number:'7.2-2' }}</td>
      <td><button (click)="deleteOperation(operation)">Delete</button> </td>
    </tr>
  </tbody>
```

Este caso de uso mantiene los datos en memoria, lo cual es muy poco fiable y sólo debe usarse con información muy volátil. Sigue esta serie para añadirle [Comunicaciones HTTP en Angular](../comunicaciones-http-en-Angular/) mientras aprendes a programar con Angular6.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>
