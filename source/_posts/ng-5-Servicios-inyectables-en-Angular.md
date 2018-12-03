---
title: Servicios inyectables en Angular
permalink: servicios-inyectables-en-Angular
date: 2018-09-11 10:44:58
tags:  
- Angular
- Servicios
- DI
- Tutorial
- Introducción
- Angular6
- Angular7
- Angular2
categories:
- [Tutorial, Angular] 
thumbnail: /css/images/angular-5_inject.png
---

![servicios-inyectables-en-Angular](/images/tutorial-angular-5_inject.png)

La presentación, la lógica y el manejo de datos son tres capas de abstracción que usamos los programadores para mantener organizado nuestro código. En Angular 6, la presentación es cosa de los componentes. **La lógica y los datos tienen su lugar en servicios compartidos**.

Para que los componentes consuman los servicios de forma controlada tenemos proveedores _inyectables_ en la librería `@angular/core` con los que realizar **la inyección de dependencias**.

<!-- more -->

Partiendo de la aplicación tal cómo quedó en [Flujo de datos entre componentes Angular](../flujo-de-datos-entre-componentes-angular/). Al finalizar tendrás una aplicación que comunica componentes entre páginas, reparte responsabilidades y gestiona claramente sus dependencias.

> Código asociado a este artículo en _GitHub_: [AcademiaBinaria/AutoBot/5-inject](https://github.com/AcademiaBinaria/autobot/tree/5-inject/)
> > Tienes una versión desplegada operativa para probar [AutoBot](https://academiabinaria.github.io/autobot/) 

# 1. Servicios

Como casi todo en Angular, **los servicios son clases TypeScript**. Su propósito es contener lógica de negocio, clases para acceso a datos o utilidades de infraestructura. Estas clases son perfectamente instanciables desde cualquier otro fichero que las importe. Pero Angular nos sugiere y facilita que usemos su sistema de inyección de dependencias.

Este sistema se basa en convenios y configuraciones que controlan la instancia concreta que será inyectada al objeto dependiente. Ahora verás cómo funciona la **[Dependency Injection](https://es.wikipedia.org/wiki/Inyecci%C3%B3n_de_dependencias) en Angular**.

## 1.1 Inyectables

La particularidad de las clases de servicios está en su decorador: `@Injectable()`. Esta función viene en el `@angular/core` e **indica que esta clase puede ser inyectada** dinámicamente a quien la demande. Aunque es muy sencillo crearlos a mano, el CLI nos ofrece su comando especializado para crear servicios. Estos son ejemplos de instrucciones para crear un *service*.

```shell
ng g s core/cars
ng g s car/engine
ng g service car/display
```

El resultado son ficheros como `cars.service.ts` con su decorador que convierte una *class* normal en algo *injectable*. Los he rellenado con un contenido como este:

```typescript
import { Injectable } from '@angular/core';
import { CARS } from './store/cars';
import { Car } from './store/models/car.model';

@Injectable({
  providedIn: 'root'
})
export class CarsService {
  private cars: Car[] = CARS;
  constructor() {}

  public getCars = () => this.cars;

  public getCarByLinkId = carId => this.cars.find(c => c.link.routerLink === carId);
}
```

Ahora tienes centralizado en este servicio la poca lógica de datos que tenemos hasta el momento. Los demás componentes la podrán utilizar.

## 1.2 Proveedor raíz

Declarar y decorar la clase no es suficiente para poder reclamarla. Necesitas **registrarla como un proveedor en algún módulo**.  En Angular 6 los servicios se autoproveen en el módoulo raíz mediante la configuración `providedIn: 'root'` de su decorador. Un ejemplo lo acabas de ver en el caso del `CarsService`.

Esto es útil y cómodo en una gran cantidad de casos. El módulo raíz es visible para toda la aplicación de forma que cualquier componente puede reclamar un servicio suyo sin problema. Excepto que el problema sea el tamaño. El módulo raíz se carga al arrancar y todas sus referencias van el *bundle* principal. Si queremos repartir el peso debemos llevar ciertos servicios al módulo funcional que los necesite. 

## 1.2 Proveedores funcionales

Servicios como el `DisplayService` o el `EngineService` sólo son reclamados por componentes del `CarModule`. En estos casos interesa que su contenido vaya en el *bundle* del módulo funcional `CarModule`. Para ello eliminamos de su decoración el `providedIn: 'root'` y los proveemos explícitamente en la propiedad `providers` de la decoración usada en `car.module.ts`


```typescript
@NgModule({
  imports: [CommonModule, CarRoutingModule, SharedModule],
  declarations: [CarComponent, IndicatorComponent, SpeedControlsComponent, BatteryRechargerComponent],
  providers: [DisplayService, EngineService]
})
export class CarModule {}
```

A partir de este momento cualquier otro servicio o componente de este módulo que los reclame **será proveído con una misma instancia** de este servicio. Fuera de este módulo no se conocen ni ocupan espacio. 

## 1.3 Singleton

Se crea un [*singleton*](https://es.wikipedia.org/wiki/Singleton) por cada módulo en el que se provea un servicio. Normalmente si el servicio es para un sólo módulo funcional se provee en este y nada más. Si va a ser compartido, entonces gana la opción de auto proveerlo en el raíz, garantizando así su disponibilidad en cualquier otro módulo de la aplicación.

Pero siempre será **una instancia única por módulo**. Si un *singleton* no es lo adecuado, entonces puedes proveer el mismo servicio en distintos módulos. De esa forma se creará una instancia distinta para cada uno. Si se provee la misma clase en dos o más módulos se genera una instancia en cada uno de ellos. Los componentes recibirán la instancia del módulo jerárquicamente más cercano.

> Incluso es posible usar el array `providers:[]` en la decoración de un componente o de otro servicio. Haciendo así aún más granular la elección de instancia. 


# 2. Dependencias

Al consumo de los servicios inyectables se le conoce como dependencia. Cada componente o servicio puede **declarar en su constructor sus dependencias** hacia servicios inyectables. El convenio exige que se especifique el tipo esperado. 

Por ejemplo, en el componente `CarComponent` teníamos incrustada toda la lógica y mantenimiento de los datos. Debe quedarse solamente con sus responsabilidades de presentación y delegar el resto en los nuevos servicios. Así queda el constructor del `car.component.ts`

```typescript
export class CarComponent implements OnInit {
  public car: Car;
  public indicators;
  constructor(
    private route: ActivatedRoute,
    private cars: CarsService,
    private display: DisplayService,
    private engine: EngineService
  ) {}
  public ngOnInit() {
    const carId = this.route.snapshot.params['carId'];
    this.car = this.cars.getCarByLinkId(carId);
    this.indicators = this.display.initilizeIndicators(this.car);
    ...
  }
  public onBrake = () => this.engine.brake(this.car);
  public onThrottle = () => this.engine.throttle(this.car);
  ...
}
```

Como ves, **el constructor no tiene otra función que la de recibir las dependencias**. Una vez construida la instancia se puede acceder a ellas a través de `this.cars` o sus semejantes. Ahora este componente ya no sabe nada sobre cómo se recuperan los datos, ni de las internalidades del motor y la lógica de presentación.

> Agregar el modificador de alcance `private` o  `public` en la declaración de argumentos hace que *TypeScript* genere una propiedad inicializada con el valor recibido. Es azúcar sintáctico para no tener que declarar la propiedad y asignarle el valor del argumento manualmente.

## 2.1 Inversión del control

Un concepto íntimamente relacionado con la inyección de depencencias es el de *Inversion of Control*. El componente dependiente expresa sus necesidades, pero es el *framework* el que en última instancia decide lo que recibirá. Vemos entonces que **el invocado cede el control al invocador**.

En Angular, el comportamiento por defecto es el de proveer un *singleton*, pero hay más opciones si se usa el objeto `provider` con `useClass` , `useValue` y `useFactory`. Por ejemplo:

```typescript
@NgModule({
  providers: [
    NormalService, 
    { provide: SameService, useClass: SameService },
    { provide: OneService, useClass: AnotherService },
    { provide: AnyService, useValue: { id: 'any', name:'compatible instance' } },
    { provide: LastService, useFactory: () => { return { id: 'something', name:'at run time' }} },
  ]
})
export class OneModule {}
```


Ya tenemos la aplicación Autobot mucho mejor estructurada, pero el almacén de datos es mejorable. Se mantienen los datos *hard-coded*, muy incómodo para actualizar; o en memoria, poco fiable y volátil. Lo más habitual es guardar y recuperar la información en un servidor *http*. Sigue esta serie para añadir [Comunicaciones HTTP en Angular](../comunicaciones-http-en-Angular/) mientras aprendes a programar con Angular6.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>
