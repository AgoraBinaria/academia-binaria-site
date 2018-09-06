---
title: Flujo de datos entre componentes Angular
permalink: flujo-de-datos-entre-componentes-angular
date: 2018-09-06 17:10:44
tags:  
- Angular
- Forms
- Tutorial
- Introducción
categories:
- [Tutorial, Angular] 
thumbnail: /css/images/angular-4_flow.png
---
![flujo-de-datos-entre-componentes-angular](/images/tutorial-angular-4_flow.png)

Los desarrollos profesionales son complejos pero con **Angular 6 tenemos soluciones de comunicación para pantallas complejas**. Mediante el desarrollo de componentes atómicos y reutilizables, Angular favorece la implementación de buenas prácticas.

Comunicar componentes llevarnos a código difícil de seguir. La librería `@angular/forms` ofrece *tuberías de comunicación* para **mantener el flujo de datos bajo control**.

<!-- more -->

Partiendo de la aplicación tal cómo quedó en [Formularios, tablas y modelos de datos en Angular](../formularios-tablas-y-modelos-de-datos-en-angular/). Al finalizar tendrás una aplicación que reparte la responsabilidad de recoger y presentar datos en componentes.

>Código asociado a este artículo en *GitHub*: [AcademiaBinaria/AutoBot/4-flow](https://github.com/AcademiaBinaria/autobot/tree/4-flow) 
> > Tienes una versión desplegada operativa para probar [AutoBot](https://academiabinaria.github.io/autobot/) 


# 1. Comunicación entre componentes de una página

Es habitual crear un componente por página. Es muy común que esa página se complique. Y la solución a la complejidad es la **división en componentes y reparto de responsabilidades**. 

Partiendo de un componente como era el `CarComponent` vemos que tenía asociadas múltiples responsabilidades: 
- Obtener los datos desde el *store* (en este caso un simple *array hard-coded*)
- Presentar la información realtiva al coche
- Responder a los eventos de aceleración y frenado del usuario conductor
- Actualizar los indicadores de velocidad y bateria

Las buenas prácticas de arquitectura de software nos obligan a repartir mejor esas responsabilidades. Para empezar vamos a diferenciar los procesos de obtención y manipulación de datos, de los de interacción y presentación de información al usuario.

>En la [implementación anterior del `CarComponent`](https://github.com/AcademiaBinaria/autobot/blob/3-data/src/app/car/car/car.component.ts) estaba todo en un único componente pues no sabíamos como repartirlo y comunicar los componentes resultantes.
>> Tampoco importaba meter todo en un único fichero (vista y controlador), pero si queremos hacerlo bien, empezaremos a repartir también en ficheros especializados (la plantilla en `.html` y el controlador en `.ts`).

Los datos han de guardarse y recuperarse en componentes distintos. Tenemos **dos estrategias** para lograrlo: tener **un único responsable de los datos, o que cada componente se encargue** de sus datos. La primera se llama controlador-presentadores.

## 1.1 Controlador y presentadores

La estrategia de un controlador y múltiples presentadores es la más adecuada para la mayor parte de las situaciones. Es la que he escogido para este ejercicio. El componente controlador maneja los datos, y los presentadores... los presentan.

Haremos que **el componente controlador** `CarComponent` sea el guardián del acceso a los datos. Mientras que **los componentes presentadores** `IndicatorComponent`, `SpeedControlComponent` y `BatteryRechargerComponent` recibirán la información para presentar y notificarán la intención de realizar cambios a su controlador.

Para eso tienes que usar dos decoradores de Angular: `@Input()` y `@Output()`.

### 1.1.1 @Input()

Para que una vista muestre datos tiene que usar directivas como `{{ value }}` asociada a una propiedad pública de la clase componente. Se supone que dicha clase es la responsable de su valor. Pero también puede **recibirlo desde el exterior**. La novedad es hacer que lo reciba vía *html*.

Empieza por decorar con `@Input()` la propiedad que quieres usar desde fuera. Por ejemplo un código como este del archivo `indicator.component.ts`.

```typescript
export class IndicatorComponent implements OnInit {
  @Input() public indicator: Indicator;
}
```

Ahora puedes enviarle datos a este componente desde el *html* de su consumidor. Por ejemplo desde `car.component.html` le puedo enviar una constante o, mucho más interesante, el valor de una variable. Incluso desde una directiva estructural como `*ngFor`.

```html
  <app-indicator *ngFor="let indicator of indicators"
                [indicator]="indicator">
  </app-indicator>
```

Y en su clase controladora tenemos el código que almacena los datos, cada vez más refindos según avanzamos en nuestro conocimiento de Angular y TypeScript. 

```typescript
export class CarComponent implements OnInit {
  public indicators = [
    { value: 0, max: 0, class: 'is-info', tags: [
        { caption: 'Speed', value: 0, class: 'is-info' },
        { caption: 'Top', value: 0, class: 'is-danger'} ]
    },
    { value: 0, max: 0, class: 'is-success', tags: [
        { caption: 'Traveled', value: 0, class: 'is-success' },
        { caption: 'Remaining',  value: 0, class: 'is-danger' } ]
    }
  ];
}
```

Estoy usando al componente de nivel inferior como un presentador; mientras que el contenedor superior actúa como controlador. Este mismo patrón puede y debe repetirse hasta **descomponer las vistas en estructuras simples** que nos eviten repeticiones absurdas en código. 

Los componentes presentadores son como clases en *POO* que se van llamando unas a otras. Puedes comprobar que el `IndicatorComponent` se construye con piezas más pequeñas como el `DataTagComponent`. Veamos un extracto del `indicator.component.html`.

```html
  <div *ngFor="let tag of indicator.tags">
    <app-data-tag [caption]="tag.caption"
                  [value]="tag.value | number:'1.0-0'"
                  [tagClass]="tag.class">
    </app-data-tag>
  </div>
```

De esta forma es fácil crear componentes reutilizables; y queda muy limpio el **envío de datos hacia abajo**. Pero, ¿y hacia arriba?.

### 1.1.2 @Output()

Los componentes de nivel inferior no sólo se dedican a presentar datos, también presentan controles. Con ellos el usuario podrá crear, modificar o eliminar los datos que quiera. Aunque no directamente; para hacerlo **comunican el cambio requerido al controlador de nivel superior**.

Por ejemplo, el componente `SpeedControlsComponent` y el `BatteryRechargerComponent` permiten acelerar, frenar y recargar la bateria. Bueno, realmente permite que el usuario diga que lo quiere hacer; los cambios se harán más arriba. Veamos lo básico del `speed-controls.component.html` antes de nada:

```html
<section>
  <button [disabled]="brakeDisabled"
      (click)="brake.next()">Brake</button>
  <button [disabled]="throttleDisabled"
      (click)="throttle.next()">Throttle</button>
</section>
```

Claramente son un par de botones que con el evento `(click)` responden a acciones del usuario. En este caso se manifiesta una intención de acelerar o frenar el coche. Pero el método del controlador no actúa directamente sobre los datos. 

> Si lo hiciera sería más difícil gestionar los cambios e imposibilitaría el uso de inmutables o técnicas más avanzadas de programación que se verán más adelante...

En su lugar, lo que hace es **emitir un evento** confiando que alguien lo reciba y actúe en consecuencia. Por ejemplo la emisión de la instrucción de frenado se realiza mediante la propiedad `brake` decorada con `@Output() public brake new EventEmitter<void>();`. Dicha propiedad será una instancia de un emisor de eventos que mediante el método `.next()` pues... emite la señal hacia arriba.

```typescript
export class ListComponent implements OnInit {
  @Output() public brake = new EventEmitter<void>();
  @Output() public throttle = new EventEmitter<void>();
}
```

Mientras tanto, **en el controlador principal la vista se subscribe al evento** `(brake)` como si este fuese un evento nativo y llama a los métodos que manipulan los datos de verdad. 

```html
  <app-speed-controls
      (brake)="onBrake()"
      (throttle)="onThrottle()">
  </app-speed-controls>
```

Las propiedades *output* también pueden enviar argumentos que serán recibidos mediante el identificador `$event` propio del framework. Un ejemplo lo tenemos en el `BatteryRechargerComponent`.

Se declara especificando el tipo del argumento en el genérico del constructor de `EventEmitter<>`.

```typescript
  @Output() public recharge = new EventEmitter<number>();
``` 
Y se envía igualmente mediante el método `.next(argumento)`

```html
  <form (ngSubmit)="recharge.next(rechargedDistance)">
  ...
  </form>
```

Para recogerlo, en el `car.component.html` se usa `$event`

```html
  <app-battery-recharger
        (recharge)="onRecharge($event)">
  </app-battery-recharger>
```
Y en la clase controladora es un argumento corriente del método de negocio.

```typescript
  public onRecharge(rechargedDistance: number) {
    this.car.remainingBattery = rechargedDistance;
  }
``` 

En el componente principal ya podemos operar con los datos. El método `onRecharge(rechargedDistance: number)` accede y modifica el valor de los indicadores y el componente principal lo notifica automáticamente hacia abajo.

De esta manera se cierra el círculo. Los componentes de bajo nivel pueden **recibir datos para ser presentados o emitir eventos para modificarlos**. El componente de nivel superior es el **único responsable de obtener y actuar** sobre los datos.

## 1.2 Múltiples controladores

Cuando las pantallas se hacen realmente complejas, empiezan a surgir **árboles de componentes** de muchos niveles de profundidad. En estas situaciones mantener un único controlador a nivel raíz es poco práctico. 

La solución en esos casos pasa porque *algunos componentes tengan su propio control de datos*. Para que esto tampoco te lleve a un caos incontrolable te enseñaré cómo resolverlo usando *Observables*. Pero eso será más adelante. 

# 2. Otras comunicaciones

## 2.1 Comunicación entre distintas páginas

En las aplicaciones hay **comunicaciones de estado más allá de la página actual**. La comunicación entre páginas es responsabilidad del `@angular/router`. Una vez activada una ruta, el sistema carga un compoente en el `<router-outlet>` correspondiente. No hay forma de comunicarse hacia *(arriba) o [abajo]* con algo desconocido. De una página a otra tampoco es problema pues la comunciación va mediante los parámetros de la *url*.

Por ejemplo el [componente `CarComponent`](https://github.com/AcademiaBinaria/autobot/blob/4-flow/src/app/car/car/car.component.ts) es capaz de recibir por parámetros una identificación de un coche. Esa información es el resultado de una acción del usuario en la patalla `/home` programada en el componente `HomeComponent`. Por tanto es una comunicación entre componentes, en la que ambos son *controladores hermanos*. 

> Desde luego habrá que mejorar el acceso y control de los datos que por ahora es muy rudimentario. Lo haremos en próximos pasos. Primero mediante  [Servicios inyectables en Angular](../servicios-inyectables-en-Angular/) y después usando [Comunicaciones HTTP en Angular](../categories/Tutorial/Angular/)

## 2.2 Comunicación entre estructuras

Otra situación habitual es **comunicar la vista de negocio activa con elementos generales** de la página. Por ejemplo podrías querer mostrar la velocidad máxima alcanzada en la barra del menú. Una vez más, el `<router-outlet>` es una barrera que impide usar el patrón controlador-presentador con su contenido dinámico.

Este tipo de comunicaciones se resuelve mediante *Observables* y merece un capítulo especial que se verá más adelante en esta serie. 

Por ahora tienes una aplicación en *Angular* que comunica datos y cambios entre componentes de una misma página. Sigue el tutorial para añadirle [Servicios inyectables en Angular](../servicios-inyectables-en-Angular/) mientras aprendes a programar con Angular6.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>