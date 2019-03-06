---
title: Vigilancia y seguridad en Angular
permalink: vigilancia-y-seguridad-en-Angular
date: 2019-03-06 13:49:27
tags:
- Angular
- http
- Observables
- Tutorial
- Introducción
- Angular7
- Angular2
categories:
- [Tutorial, Angular]
thumbnail: /css/images/angular-7_watch.png
---

![vigilancia-y-seguridad-en-Angular](/images/tutorial-angular-7_watch.png)

La vigilancia de los datos y la información en tiempo real al usuario son dos pilares del desarrollo con Angular e el lado del navegador. La seguridad de los datos es una responsabilidad compartida entre el servidor y el cliente. En **Angular** usaremos los _interceptores_ para detectar intrusos y enviar credenciales.

Veremos ambos aspectos del desarrollo, pues están muy relacionados con la programación asíncrona y el dominio de los observables. Sentaremos las bases para unas **comunicaciones seguras y fluidas en Angular**.

<!-- more -->

Partiendo de la aplicación tal cómo quedó en [Comunicaciones http en Angular](../comunicaciones-http-en-Angular/). Al finalizar tendrás una aplicación que comunica a los usuarios cualquier información relevante y que gestiona de forma centralizada las respuestas de un servicio REST.

> Código asociado a este artículo en _GitHub_: [AcademiaBinaria/angular-board/7-watch](https://github.com/AcademiaBinaria/angular-board/tree/master/src/app/7-watch/notifications)
>
> > Tienes una versión desplegada operativa para probar [Angular Board](https://academiabinaria.github.io/angular-board/)

# 1. Observables para monitorizar datos

Hemos visto varias técnicas para comunicar información dentro de una aplicación Angular. Empezamos por conocer el flujo entre componentes de una misma rama del DOM. También enviamos datos en los parámetros de una ruta. Y podemos usar un servicio común para guardar información compartida. Pero en este caso, ¿Cuándo se actualiza? ¿Cómo saber si ha cambiado?. Lo resolveremos con Observables.

Para ilustrar este tema vamos a crear un sencillo sistema de notificaciones para informar al usuario. Para ello creamos un módulo para los propósitos de este laboratorio.

```console
ng g m notifications --routing true
ng g c notifications/sender
ng g c notifications/receiver
```

Lo apuntamos al enrutador global `app-routing.module.ts` y asignamos las rutas locales

```typescript
{
  path: 'notifications',
  loadChildren: './notifications/notifications.module#NotificationsModule'
},
```
```typescript
const routes: Routes = [
  {
    path: 'sender',
    component: SenderComponent
  },
  {
    path: 'receiver',
    component: ReceiverComponent
  },
  {
    path: '**',
    redirectTo: 'sender'
  }
];
```

Por último un enlace en el menú principal `header.component.html` y ya estamos listos para enviar y recibir desde dos componente desacoplados.

```html
<a routerLink="notifications" class="button">Notifications</a>
```

Pero antes un poco más de observables.

## 1.1 Productores de observables

La librería RxJS es enorme y Angular hace un uso extenso de ella. En este tutorial se ha visto desde el punto de vista del consumidor. Es decir, nos hemos suscrito a fuentes observables. Para avanzar tendremos que poder emitir, o mejor dicho producir, información.

### Of from interval

Los constructores más sencillos de la librería son funciones que emiten valores estáticos o secuencias a intervalos regulares. Para familiarizarte con ellos te propongo que juegues con código como este:

```typescript
value$ = of(new Date().getMilliseconds());
value$.subscribe(r=> console.log(r));
stream$ = from([1, 'two', '***']);
stream$.subscribe(r=> console.log(r));
list$ = of(['N', 'S', 'E', 'W']);
list$.subscribe(r=> console.log(r));
```

### Subject y BehaviorSubject

Los anteriores constructores se basan en datos estáticos. Resuelvan algunas situaciones, pero necesitamos algo que emita cambios dinámicos. Y eso se resuelve con los _Subjects_, una especie de emisores temáticos a los que suscribirse.

Hay varios tipos pero para empezar nos vamos a fijar en dos: el `Subject()` y el `BehaviorSubject(initialData)`. La diferencia es que el primero sólo emite las cosas según ocurren. Si alguien se suscribe tarde no conocerá el pasado. Esto suele generar problemas de sincronización. El _Behavior_ en cambio notifica el último valor conocido a cualquiera que se suscriba. De esa forma no importa si te suscribes antes o después de la obtención de un dato.

Juega con el siguiente ejemplo:

```typescript
const data = {name:'', value:0};

const need_sync$ = new Subject<any>();
// on time
need_sync.subscribe(r=> console.log(r));
need_sync.next(data);
// too late
need_sync.subscribe(r=> console.log(r));

const no_hurry$ = new BehaviorSubject<any>(this.data);
// its ok
no_hurry.subscribe(r=> console.log(r));
no_hurry.next(data);
// its also ok
no_hurry.subscribe(r=> console.log(r));
```

## 1.2 Un Store de notificaciones

Usaremos el `BehaviorSubject` como notificador principal entre componentes de Angular. Por ejemplo podemos montar un sistema de notificaciones sencillo. Empecemos por crear un servicio:

```console
ng g s notifications/notificationsStore
```

> Para adaptarnos a la nomenclatura usada por patrones de gestión de estado más avanzados como es Redux, usaré el siguiente convenio: _Store_ como almacén, _select$()_ como publicador de cambios observable y _dispatch_ como encargado de procesar una acción de cambio de estado.

```typescript
export class NotificationsStoreService {
  private notifications = [];

  private notifications$ = new BehaviorSubject<any[]>([]);

  constructor() {}

  public select$ = () => this.notifications$.asObservable();

  public dispatchNotification(notification) {
    this.notifications.push(notification);
    this.notifications$.next([...this.notifications]);
  }
}
```

Este servicio es la implementación más sencilla posible de un gestor de estados. Cabe destacar que :

- Mantienen el estado privado para evitar manipulaciones
- Recibe de forma controlada las acciones de cambio
- Emite clones del estado
- Expone observables para que se suscriban los interesados.


## 1.3 Desacoplados pero conectados

Una vez que hemos centralizado el control de cambios de una parte de la aplicación, es hora de que lo usen los componentes o servicios involucrados. Sólo necesitan recibir la instancia vía dependencia. No hay más acoplamiento entre emisores y receptores.

### Emisión

Veamos un ejemplo, un tanto forzado,  consistente en dos componentes que se comunican sin conocerse. Vista con un formulario para enviar mensajes

```html
<h2>
  Notes sender
</h2>
<form>
  <fieldset>
    <section>
      <label for="note">Note</label>
      <input name="note"
             [(ngModel)]="note" />
    </section>
  </fieldset>
  <button (click)="send()">Send</button>
</form>
<a [routerLink]="['../receiver']">Go to receiver</a>
```

La parte interesante está en el controlador. Dependencia y uso del servicio del almacén de notificaciones

```typescript
export class SenderComponent implements OnInit {
  public note = '';

  constructor(private notificationsStore: NotificationsStoreService) {}

  ngOnInit() {}

  public send() {
    this.notificationsStore.dispatchNotification(this.note);
  }
}
```

### Recepción

La recepción es igual de sencilla. En la vista pondremos un listado de notificaciones que se alimenta de un array emitido por un observable.

```html
<h2>
  Notes receiver
</h2>
<ul>
  <li *ngFor="let note of notes$ | async">{{ note | json }}</li>
</ul>
<a [routerLink]="['../sender']">Go to sender</a>
```

Y en el controlador reclamamos la misma dependencia para el uso del servicio del almacén de notificaciones.

```typescript
export class ReceiverComponent implements OnInit {
  public notes$;

  constructor(private notificationsStore: NotificationsStoreService) {}

  ngOnInit() {
    this.notes$ = this.notificationsStore.select$();
  }
}
```

Es importante recalcar que no importa el orden de suscripción. Estos dos componentes podrían _vivir_ en módulos distintos, verse en la misma página o inicializarse en cualquier orden... El receptor se entera siempre de todos los cambios; y además recibe el último estado conocido nada más suscribirse.

# 2. Interceptores para gestionar errores

W.I.P.



> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>
