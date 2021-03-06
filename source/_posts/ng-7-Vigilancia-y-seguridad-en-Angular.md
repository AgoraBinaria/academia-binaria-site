---
title: Vigilancia y seguridad en Angular
permalink: vigilancia-y-seguridad-en-Angular
date: 2020-04-19 11:49:27
tags:
- Angular
- http
- Observables
- Tutorial
- Introducción
- Angular9
- Angular2
categories:
- [Tutorial, Angular]
thumbnail: /css/images/angular-7_watch.png
---

![vigilancia-y-seguridad-en-Angular](/images/tutorial-angular-7_watch.png)

La **vigilancia** de los datos y la información en tiempo real al usuario son dos pilares del desarrollo con Angular en el lado del navegador. La **seguridad** de los datos realmente es una responsabilidad compartida entre el servidor y el cliente.

Veremos ambos aspectos del desarrollo, pues están muy relacionados con la programación asíncrona y el dominio de los observables. Sentaremos las bases para unas **comunicaciones seguras y fluidas en Angular**.

<!-- more -->

Partiendo de la aplicación tal como quedó en [Comunicaciones http en Angular](../comunicaciones-http-en-Angular/). Al finalizar tendrás una aplicación que comunica a los usuarios cualquier información relevante y que gestiona de forma centralizada las respuestas de un servicio REST.

> Código asociado a este tutorial en _GitHub_: [AcademiaBinaria/angular-basic](https://github.com/AcademiaBinaria/angular-basic/)

# 1. Observables para monitorizar datos

Hemos visto varias técnicas para comunicar información dentro de una aplicación Angular. Empezamos por conocer el [flujo entre componentes](../flujo-de-datos-entre-componentes-angular/) de una misma rama del DOM. También enviamos datos en los [parámetros de una ruta](../paginas-y-rutas-angular-spa/). Y obviamente podemos usar [un servicio común](../servicios-inyectables-en-Angular/) para guardar información compartida. Pero en este caso, ¿Cuándo se actualiza? ¿Cómo saber si ha cambiado?. Lo resolveremos con `Observables`.

Para ilustrar este tema vamos a crear una web que muestra los datos de los próximos 5 lanzamientos espaciales. Y después un sencillo sistema de notificaciones que simula las comunicaciones de la Nasa. Empezaremos creando un módulo para los propósitos de este laboratorio.

```console
ng g m rockets --route rockets --module app-routing.module
```

Y en el componente tendremos que usar llamadas http pero sin suscripciones manuales. En su lugar nos quedamos con el observable y lo consumimos desde la vista con una de mis pipes preferidas: ` | async`

```typescript
  private rocketsApi = 'https://launchlibrary.net/1.4/';
  public nextLaunches$: Observable<any> = null;

  constructor(private httpClient: HttpClient) {}

  ngOnInit() {
    this.getNextLaunches();
  }

  private getNextLaunches() {
    const url = this.rocketsApi + 'launch/next/5';
    this.nextLaunches$ = this.httpClient.get<any>(url);
  }
```

En la plantilla accedemos al observable. La tubería se suscribe y retorna el resultado cuando se reciba. El _pipe_ `json` nos permite ver el resultado.

```html
<h3> Next 5 rocket launches </h3>
<pre>{{ nextLaunches$ | async | json }}</pre>
```

Y eso es todo. Casi nunca tendrás que suscribirte en código. Deja que ese trabajo sucio lo haga el _pipe async_.
Pero, siempre hay un pero, rara vez los datos te vendrán tal cual los necesitas. Casi seguro que en ocasiones habrá que filtrarlos, ordenarlos y sobre todo aplicarles alguna trasformación. Para ello, antes tienes que saber un poco más de observables.

# 2 Observables

La librería [RxJS](https://www.learnrxjs.io/) es enorme y Angular hace un uso extenso de ella. En este tutorial [se ha visto desde el punto de vista del consumidor](../comunicaciones-http-en-Angular). Es decir, nos hemos suscrito a fuentes observables. Para avanzar tendremos que poder emitir, o mejor dicho producir, información.

## 2.1 Productores simples of y from

Los constructores más sencillos de la librería son **funciones que emiten valores estáticos** o secuencias a intervalos regulares. Para familiarizarte con ellos te propongo que juegues con código como este:

```typescript
value$ = of(new Date().getMilliseconds());
value$.subscribe(r=> console.log(r));
stream$ = from([1, 'two', '***']);
stream$.subscribe(r=> console.log(r));
list$ = of(['N', 'S', 'E', 'W']);
list$.subscribe(r=> console.log(r));
```

## 2.2 Operadores simples map y tap

Tratamos a los observables como un flujo, un _stream_ de datos; algo similar a un riachuelo de gotas de agua. A cada dato recibido le podemos aplicar funciones; sería como aplicar algún tratamiento al agua del riachuelo. Vamos a empezar con dos categorías de acciones: las que transforma en contenido, y la que tienen efectos secundarios.

Para ello disponemos de unas funciones de primer orden llamadas operadores. Estas funciones reciben como argumento a otras funciones en las que tú programas las acciones de transformación o efecto.

El operador de transformación se llama `map()` y el de efectos se llama `tap()`. Ambos se pueden usar cuantas veces se quiera y en cualquier orden. Sólo hay que meterlos dentro de una función superior. La función `pipe()`; que da nombre al proceso: entubar el stream.

```typescript
numbers$ = from([1, 2, 3, 4]);
doubles$ = numbers$
  .pipe(
    map(x=> 2 * x ),
    tap(x=Z console.log(x)) // 2 4 6 8
    );
```


Apliquemos esto a los cohetes. Por ejemplo editando y transformando los datos recibidos desde el api:

```typescript
private getNextLaunches() {
  const url = this.rocketsApi + 'launch/next/5';
  this.nextLaunches$ = this.httpClient.get<any>(url).pipe(
    map(apiData => apiData.launches),
    map(launchesArray =>
      launchesArray.map(launch => ({
        name: launch.name,
        status: launch.status,
        scheduled: launch.net,
      }))
    ),
    map(customLaunches =>
      customLaunches.map(launch => ({
        ...launch,
        statusColor: launch.status === 1 ? 'green' : 'red',
      }))
    ),
    tap(rockets => console.log('num rockets:' + rockets.length)),
    tap(() => console.log('ha pasado algo')),
    map(()=>{
      const a =2;
    })
  );
}
```

De esta forma la _template_ ya recibe los datos listos para presentar de una manera más vistosa y limpia.


```html
<ul *ngIf="nextLaunches$ | async as nextLaunches; else loading">
  <li *ngFor="let launch of nextLaunches"
      [style.color]="launch.statusColor">
    <b>{{ launch.name }}</b><i>{{ launch.scheduled }}</i>
  </li>
</ul>
<ng-template #loading>
  <i>Loading rocket data... </i>
</ng-template>
```

# 3 Un store de notificaciones

## 3.1 Subject y BehaviorSubject

Los anteriores constructores se basan en datos estáticos. Resuelvan algunas situaciones, pero necesitamos algo que emita **cambios dinámicos**. Y eso se realiza con los _Subjects_, una especie de emisores temáticos a los que suscribirse.

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

## 3.2 Un Store de notificaciones

Usaremos el `BehaviorSubject` como notificador principal entre componentes de Angular. Por ejemplo podemos montar un sistema de notificaciones sencillo. Empecemos por crear un servicio:

```bash
ng g m rockets/nasa --route nasa --module rockets/rockets-routing.module
ng g c rockets/nasa/houston
ng g c rockets/nasa/florida
```

  Este servicio es la implementación más sencilla posible de un gestor de estados. Cabe destacar que :

```typescript
export class MissionsComunicationService {
  private messages: object[] = [{ icon: '👩‍🚀', subject: 'Crew on board' }];
  private messages$ = new BehaviorSubject<any[]>(this.messages);

  constructor() {}

  public getMessages$ = () => this.messages$.asObservable();

  public sendMessage = (message: object) => {
    this.messages.push(message);
    this.messages$.next(this.messages);
  };
}
```

- Mantienen el estado privado para evitar manipulaciones
- Recibe de forma controlada las acciones de cambio
- Emite clones del estado
- Expone observables para que se suscriban los interesados.

> Podría adaptar a la nomenclatura usada por patrones de gestión de estado más avanzados como es **Redux**, y usar el siguiente convenio: _Store_ como almacén, _select$()_ como publicador de cambios observable y _dispatch_ como encargado de procesar una acción de cambio de estado.

## 3.3 Desacoplados pero conectados

Una vez que hemos centralizado el control de cambios de una parte de la aplicación, es hora de que lo usen los componentes o servicios involucrados. Solo necesitan recibir la instancia vía dependencia. **No hay más acoplamiento entre emisores y receptores.**

### Emisión

Veamos un ejemplo, un tanto forzado,  consistente en dos componentes que se comunican sin conocerse. Esta es la vista con un formulario para enviar mensajes

```html
<h4>Houston mission control</h4>
<p><button> Start count down 🏁</button></p>
<p><button> Ignition 🔥</button></p>
<p><button> Abort 🛑</button></p>
```

La parte interesante está en el controlador. Dependencia y uso del servicio del almacén de notificaciones

```typescript
export class HoustonComponent implements OnInit {
  constructor(private missionsComunicationService: MissionsComunicationService) {}

  ngOnInit(): void {}

  onStartClick() {
    this.missionsComunicationService.sendMessage({ icon: '🏁', subject: 'Start count down' });
  }
  onIgnitionClick() {
    this.missionsComunicationService.sendMessage({ icon: '🔥', subject: 'Ignition' });
  }
  onAbortClick() {
    this.missionsComunicationService.sendMessage({ icon: '🛑', subject: 'Abort' });
  }
}
```

### Recepción

La recepción es igual de sencilla. En el controlador reclamamos la misma dependencia para el uso del servicio del almacén de notificaciones.

```typescript
export class FloridaComponent implements OnInit {
  messages$: Observable<any>;

  constructor(private missionsComunicationService: MissionsComunicationService) {}

  ngOnInit(): void {
    this.messages$ = this.missionsComunicationService.getMessages$();
  }
}
```

En la vista pondremos un listado de notificaciones que se alimenta de un array emitido por un observable.

```html
<h4>Florida launch pad</h4>
<p>📋 Mission log</p>
<ul>
  <li *ngFor="let message of messages$ | async">{{ message }}</li>
</ul>
```

Es importante recalcar que **no importa el orden de suscripción**. Estos dos componentes podrían _vivir_ en módulos distintos, verse en la misma página o inicializarse en cualquier orden... El receptor se entera siempre de todos los cambios; y además recibe el último estado conocido nada más suscribirse.



Tenemos ahora a nuestro usuario puntualmente informado de todo lo que sucede. Hemos utilizado **patrones de arquitectura de software** como el observable y la inversión del control. El resultado es una serie de componentes y servicios poco acoplados que intercambian información en tiempo real.

Pero hay temas más avanzados con los que continuar. Sigue esta serie para introducir más temas de seguridad y crear tus [formularios reactivos con Angular](../formularios-reactivos-con-Angular/) mientras aprendes a programar con Angular. Todos esos detalles se tratan en el [curso básico online](https://www.trainingit.es/curso-angular-basico/?promo=angular.builders) que imparto con TrainingIT o a medida para tu empresa.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>
