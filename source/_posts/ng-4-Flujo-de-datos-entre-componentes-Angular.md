---
title: Flujo de datos entre componentes Angular
permalink: flujo-de-datos-entre-componentes-angular
date: 2017-11-20 17:10:44
tags:  
- Angular
- Angular5
- Angular2
- Forms
- Tutorial
- Introducción
categories:
- [Tutorial, Angular] 
thumbnail: /css/images/angular-5_4_flow.png
---
![Tutorial Angular5 4-Flow](/images/tutorial-angular-5_4_flow.png)

Los formularios profesionales son complejos y **Angular ofrece soluciones de comunicación para pantallas complejas**. Favorece la implementación de buenas prácticas mediante el desarrollo de componentes atómicos y reutilizables.

Pero comunicar componentes no es tarea fácil y puede generar código difícil de seguir. La librería `@angular/forms` ofrece *tuberías de comunicación* para **mantener el flujo de datos bajo control**.

<!-- more -->

Partiendo de la aplicación tal cómo quedó en [Formularios, tablas y modelos de datos en Angular](../formularios-tablas-y-modelos-de-datos-en-angular/). Al finalizar tendrás una aplicación que reparte la responsabilidad de recoger y presentar datos en dos componentes.

>Código asociado a este artículo en *GitHub*: [AcademiaBinaria/angular5/4-flow](https://github.com/AcademiaBinaria/angular5/tree/master/4-flow/cash-flow) 


# 1. Comunicación entre componentes de una página

Es habitual crear un componente por página. Es muy común que esa página se complique. Y la solución a la complejidad es la **división en componentes y reparto de responsabilidade**s. 

Partiendo de un componente como era el `OperationsComponent` vemos que tenía asociadas dos tareas: recoger en un formulario y mostrar en una tabla los datos de operaciones financieras. Para repartir la responsabilidad creamos un componente, el `NewComponent`, para el formulario y otro, el `ListComponent`, para la tabla.

>En la [implementación anterior del `NewComponent`](https://github.com/AcademiaBinaria/angular5/blob/master/3-data/cash-flow/src/app/views/operations/new.component.ts) estaba todo pues no sabíamos como llevarlo a otro sitio y comunicar los componentes.

Los datos han de guardarse y recuperarse en componentes distintos; tenemos **dos estrategias** para lograrlo. Tener **un único responsable o que cada componente se encargue** de sus datos.

## 1.1 Controlador y presentadores

La estrategia de un controlador y múltiples presentadores es la más adecuada para la mayor parte de las situaciones. Es la que he escogido para este ejercicio.

Se basa en que **el componente contenedor** `OperationsComponent` sea el **guardián del acceso** a los datos. Mientras que **los componentes presentadores** `NewComponent` y `ListComponent` **recibirán la información y notificarán los cambios** a su padre controlador.

Para eso tienes que usar dos decoradores de Angular: `@Input()` y `@Output()`.

### 1.1.1 @Input()

Para que una vista muestre datos tiene que usar directivas como `{ {numberOfOperations} }` asociada a una propiedad pública de la clase componente. Se supone que dicha clase es la responsable de su valor. Pero también puede **recibirlo desde el exterior**. La novedad es hacer que lo reciba vía *html*.

Empieza por decorar con `@Input()` la propiedad que quieres usar desde fuera. Por ejemplo un código como este del archivo `list.component.ts`.

```typescript
export class ListComponent implements OnInit {
  @Input() public numberOfOperations = 0;
  @Input() public operations: Operation[] = [];
}
```

Ahora puedes enviarle datos a este componente desde el *html* de su consumidor. Por ejemplo desde `operations.component.ts` le puedo enviar una constante o, mucho más interesante, el valor de una variable.

```html
<cf-list 
  [numberOfOperations]="numberOfOperations" 
  [operations]="operations" >
</cf-list>
```

Y en su clase controladora tenemos el código que almacena los datos. 

```typescript
export class OperationsComponent implements OnInit {
  public numberOfOperations = 0;
  public operations: Operation[] = [];
}
```

Estoy usando al componente de nivel inferior como un presentador; mientras que el contenedor superior actúa como controlador. De esta forma es fácil y queda muy limpio el **envío de datos hacia abajo**. Pero, ¿y hacia arriba?.

### 1.1.2 @Output()

Los componentes de nivel inferior no sólo se dedican a presentar datos. También pueden crearlos, modificarlos o eliminarlos. Aunque no directamente; para hacerlo **comunican el cambio requerido al controlador de nivel superior**.

Por ejemplo, el mismo componente `ListComponent` además de mostrar operaciones en una tabla permite borrar un registro. Bueno, realmente permite que el usuario diga que quiere borrar un registro. En su *html* tiene algo así:

```html
<tr *ngFor="let operation of operations">
  <td>{{ operation.description }}</td>
  <td>{{ operation.kind }}</td>
  <td>{{ operation.amount | number:'7.2-2' }}</td>
  <td><button (click)="deleteOperation(operation)">Delete</button></td>
</tr>
```

Claramente el botón con el evento `(click)="deleteOperation(operation)"` manifiesta una intención de borrar el registro. Pero el método del componente no actúa directamente con el array de datos. 

>Si lo hiciera haría difícil gestionar los cambios e imposibilitaría el uso de inmutables o técnicas más avanzadas de programación..

En su lugar, lo que hace es **emitir un evento**, confiando que alguien lo reciba y actúe en consecuencia. La emisión se realiza mediante el decorador `@Output() public delete`, sobre una propiedad que es un emisor de eventos *tipado*, `new EventEmitter<Operation>();`. El método `deleteOperation(operation)`, es un delegado al que llama la vista y usa dicho emisor para... ejem, emitir la señal hacia arriba.

```typescript
export class ListComponent implements OnInit {
  @Output() public delete = new EventEmitter<Operation>();

  public deleteOperation(operation: Operation) {
    this.delete.emit(operation);
  }
}
```

Mientras tanto, **en el controlador principal la vista se subscribe al evento** `(delete)` como si este fuese un evento nativo. La instrucción que se ejecuta hace uso del argumento recibido en el identificador `$event` estándar del framework.

```html
<cf-list 
  [numberOfOperations]="0" 
  [operations]="operations" 
  (delete)="deleteOperation($event)" >
</cf-list>
```

En el componente principal ya podemos operar con los datos. El método `deleteOperation(operation: Operation)` accede y modifica el valor del array `operations`. Cuando dicho array cambia en el componente principal lo notifica automáticamente hacia abajo; de nuevo hacia la lista.

```typescript
export class OperationsComponent implements OnInit {
  public numberOfOperations = 0;
  public operations: Operation[] = [];

  public deleteOperation(operation: Operation) {
    const index = this.operations.indexOf(operation);
    this.operations.splice(index, 1);
    this.numberOfOperations = this.operations.length;
  }
}
```

De esta manera se cierra el círculo. Los componentes de bajo nivel pueden **recibir datos para ser presentados o emitir eventos para modificarlos**. El componente de nivel superior es el **único responsable de obtener y actuar** sobre los datos.

## 1.2 Múltiples controladores

Cuando las pantallas se hacen realmente complejas, empiezan a surgir **árboles de componentes** de muchos niveles de profundidad. En estas situaciones mantener un único controlador a nivel raíz es poco práctico. 

La solución en esos casos pasa porque **algunos componentes tengan su propio control de datos**. Para que esto tampoco te lleve a un caos incontrolable te enseñaré cómo resolverlo usando *Observables*. Pero eso será más adelante. 

# 2. Otras comunicaciones

## 2.1 Comunicación entre distintas páginas

En las aplicaciones hay **comunicaciones de estado más allá de la página actual**. La comunicación entre páginas es responsabilidad del `@angular/router`.

En el [estado actual del componente `ItemComponent`](https://github.com/AcademiaBinaria/angular5/blob/master/3-data/cash-flow/src/app/views/operations/item.component.ts) es capaz de recibir por parámetros una identificación de operación. Pero no tiene acceso al array de datos y por tanto no los puede mostrar ni interactuar con ellos.

Desde luego necesita convertirse en un controlador, pero antes habrá que **bajar los datos a un nivel compartido entre páginas**. Lo haremos en próximos pasos. Primero mediante  [Servicios inyectables en Angular](../servicios-inyectables-en-Angular/) y después usando [Comunicaciones HTTP en Angular](../categories/Tutorial/Angular/).

## 2.2 Comunicación entre estructuras

Otra situación habitual es **comunicar la vista de negocio activa con elementos generales** de la página. Por ejemplo podrías querer mostrar el contador o un balance de operaciones en la barra del menú.

Este tipo de comunicaciones también se resuelve mediante *Observables* y merece un capítulo especial. Por ahora tienes una aplicación en *Angular* que comunica datos y cambios entre componentes de una misma página. Sigue esta serie para añadirle [Servicios inyectables en Angular](../servicios-inyectables-en-Angular/) mientras aprendes a programar con Angular5.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>