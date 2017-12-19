---
title: Comunicaciones http en Angular
permalink: comunicaciones-http-en-Angular
date: 2017-12-18 11:06:00
tags:  
- Angular
- Angular5
- http
- Tutorial
categories:
- [Tutorial, Angular] 
thumbnail: /css/images/angular-5_6_http.png
---

![Tutorial Angular5 6-http](/images/tutorial-angular-5_6_http.png)

Las comunicaciones http son una pieza fundamental del desarrollo web, y en Angular siempre han sido fáciles y potentes. ¿Siempre?, bueno cuando apareció Angular 2 echábamos en falta algunas cosillas. Pero con la versión actual **consumir un servicio REST vuelve a ser cosa de niños**.

Claro que para ello tendremos que jugar con los _observables_ y los servicios de la librería `@angular/common/http` con los que realizar **comunicaciones asíncronas en Angular**.

<!-- more -->

Partiendo de la aplicación tal cómo quedó en [Servicios inyectables en Angular](../servicios-inyectables-en-Angular/). Al finalizar tendrás una aplicación que almacena y recupera los datos consumiendo un servicio REST.

> Código asociado a este artículo en _GitHub_: [AcademiaBinaria/angular5/6-http](https://github.com/AcademiaBinaria/angular5/tree/master/6-http/cash-flow)
> El servicio REST se encuentra en _GitHub_: [AcademiaBinaria/ApiBase](https://github.com/AcademiaBinaria/ApiBase)

# 1. El servicio HttpClient

La librería `@angular/common/http` trae el módulo `HttpClientModule` con el servicio inyectable `HttpClient` que debes reclamar como dependencia en tus propios constructores.

En el fichero `operations.service.ts` tienes el código que reclama la dependencia y la configura con una ruta base obtenida de la configuración de `environment`. Queda algo así:

```typescript
export class OperationsService {
  private url = environment.apiUrl + "pub/items/";

  constructor(private http: HttpClient) {}
}
```

A partir de este momento sólo queda invocar los métodos REST en la propiedad `this.http`.

## 1.1 Métodos REST

Para cada verbo _http_ tenemos su método en el servicio `HttpClient`. Su primer parámetro será la url a la que invocar. Los métodos de envío reciben la carga en el segundo argumento, y la envían correctamente como objetos _JSON_.

Un ejemplo sencillo lo tienes en el servicio `OperationsService`.

```typescript
 public getOperationsList$(): Observable<Operation[]> {
   return this.http.get<Operation[]>(this.url);
 }
 public getOperationById$(id: string): Observable<Operation> {
   return this.http.get<Operation>(this.url + id);
 }
 public saveOperation$(operation: Operation): Observable<any> {
   return this.http.post(this.url, operation);
 }
 public deleteOperation$(operation: Operation): Observable<any> {
   return this.http.delete(this.url + operation._id);
 }
```

> Cada método de negocio, configura la llamada de infraestructura; parece poca cosa. Podría ser un buen sitio para validar la información antes de ser enviada, o quizás agrupar varias llamadas de red para una misma operación de negocio.

# 2 Observables

Las comunicaciones entre navegadores y servidores son varios órdenes de magnitud más lentas que ls operaciones en memoria. Por tanto deben realizarse de manera asíncrona para garantizar una buena experiencia al usuario.

Esta experiencia no siempre fue tan buena para el programador. Sobre todo con las primeras comunicaciones _AJAX_ basadas en el paso de funciones _callback_. La aparición de las _promises_ mejoró la claridad del código, y ahora con los _Observables_ tenemos además una gran potencia para manipular la información asíncrona.

> El patrón `Observable` fue implementado por Microsoft en la librería [_Reactive Extensions_](http://reactivex.io/intro.html) más conocida como `RxJs`. El equipo de Angular decidió utilizarla para el desarrollo de las comunicaciones asíncronas. Esta extensa librería puede resultar intimidante en un primer vistazo. Pero con muy poco conocimiento puedes programar casi todas las funcionalidades que se te ocurran.

Lo primero es importar el código, esto se hace forma similar a cualquier otra clase o función. Para empezar basta con `import { Observable } from "rxjs/Observable";`.

Esta es una clase genérica, donde sus instancias admiten la manipulación interna de tipos concretos. Por eso ves en el ejemplo que algunas funciones retornan `: Observable<Operation>`, o si no saben que tipo esperar se conforman con `: Observable<any>`.

En cualquier caso, toda operación asíncrona retornará una instancia observable a la cual habrá que subscribirse para recibir los datos, o los errores, cuando termine. Eso debes hacerlo en los componentes que consuman el servicio, no en el propio servicio.

## 2.1 Lectura

En `operations.component` están las llamadas y las suscripciones necesarias. Por ejemplo el método `refreshData()` realiza llamadas y se suscribe para conocer los resultados. Cada suscripción es un _callback_ al que habrá que pasarle el contexto mediante `.bind(this)`.

```typescript
private refreshData() {
  this.message = `Refreshing Data`;
  this.fullError = null;
  this.operationsService
    .getOperationsList$()
    .subscribe(this.showOperations.bind(this), this.catchError.bind(this));
  this.operationsService
    .getNumberOfOperations$()
    .subscribe(this.showCount.bind(this), this.catchError.bind(this));
}
private showOperations(operations: Operation[]) {
  this.operations = operations;
  this.message = `operations Ok`;
}
private showCount(data: any) {
  this.numberOfOperations = data.count;
  this.message = `count Ok`;
}
private catchError(err) {
  if (err instanceof HttpErrorResponse) {
    this.message = `Http Error: ${err.status}, text: ${err.statusText}`;
  } else {
    this.message = `Unknown error, text: ${err.message}`;
  }
  this.fullError = err;
}
```

El método `subscribe` recibe hasta tres argumentos (ok, error, fin) donde colocar funciones receptoras para cada tipo de evento que ocurra. Sólo el primero es obligatorio, y es en el que recibes la información directamente desempaquetada. En el segundo, normalmente pondrás lógica para responder ante códigos de error devueltos por el servidor. Es opcional porque hay técnicas para gestionarlos de manera centralizada pero en este ejemplo te muestro con detalle cómo tratar y analizar los eventos de error.

> Recordatorio para novatos: Es importante comprender la naturaleza asíncrona de estas operaciones. El código de las funciones subscritas se ejecutará en el futuro, no de una manera secuencial.

## 2.2 Escritura

Si lo que quieres es enviar objetos a un servidor, por ejemplo mediante el verbo _POST_, sólo tienes que pasarle la _payload_ al método de negocio y suscribirte a la respuesta.

```typescript
public saveOperation(operation: Operation) {
 this.operationsService
   .saveOperation$(operation)
   .subscribe(data => this.refreshData());
}
```

> De nuevo, fíjate como refrescamos los datos una vez recibida la respuesta. Hacerlo antes podría dar lugar a respuestas incongruentes. En este caso no es la respuesta en sí lo que interesa, sino el hecho de haya terminado bien.

# 3 Interceptores

Angular ha incorporado recientemente el concepto de interceptor que había funcionado muy bien en AngularJS. Ahora los interceptores son clases con métodos que interceptan (de ahí su nombre) todas las peticiones http realizadas. En esos métodos puedes poner lógica que modifique, cancele o simplemente controle el ciclo petición/respuesta de cada llamada.

## 3.1 Implementación de la interfaz

Aprovechando el **TypeScript** y sus características de programación orientada a objetos, en **Angular** han optado por obligarnos a cumplir _interfaces_ y como contraparte al cumplir ese contrato invocan a nuestro código en circunstancias controladas. ¿Cómo se hace?.

Para empezar hay que crear un servicio inyectable en un módulo general o en el raíz. Yo he creado el `CatchInterceptorService` en `lib/catch-interceptor.service.ts`. Su propósito será capturar las respuestas y gestionar de forma centralizada los errores que se obtengan en un único lugar. Como cualquier otro inyectable habrá que proveerlo en un módulo, yo lo hice en el raíz. Si abres el `app.module.ts` verás un sistema de aprovisionamiento complejo.

```typescript
providers: [
  {
    provide: HTTP_INTERCEPTORS,
    useClass: CatchInterceptorService,
    multi: true
  }
];
```

La dependencia que se solicitará es `HTTP_INTERCEPTORS`, y cuando alguien lo haga (son los servicios de http de Angular a bajo nivel) le será inyectada una instancia de la clase `CatchInterceptorService`.

> Es decir, tu clase interceptora será usada sin que el que la reclame conozca siquiera su nombre. Pero esto es la clave de la inversión de control en la inyección de dependencias.

El parámetro `multi: true`, indica que puedes crear tantas clases de interceptación como quieras. Esto permite tener múltiples interceptores especializados.

## 3.2 El método de interceptación

Al implementar la interfaz `HttpInterceptor` estás obligado a crear un método con una firma como esta: `public intercept(req, next)`. Este será invocado para cada llamada que hecha con `httpClient`. A esta función se la conoce como _función interceptora_ y ya ves que tiene unos argumentos y un tipo de retorno bien definidos... y algo complejos a primera vista. Veamos una implementación mínimalista.

```typescript
public intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
  return next.handle(req);
}
```

Lo que dice es que recibe un puntero a la petición en curso, `req: HttpRequest<any>`, y otro a la siguiente clase que la procese,`next: HttpHandler`. También obliga a devolver un observable; lógico porque esto es en último término a lo que te suscribes en tu código de negocio. Afortunadamente el método `handle(req: HttpRequest<any>)` de cualquiera que sea la siguiente clase procesadora retorna un observable. Y así se da continuidad al flujo de la llamada.

## 3.3 Los operadores observables

Además de tipos de datos como `Observable<any>` con métodos clave como `.subscribe(ok, err, end)`, la librería _RxJs_ viene cargadita de [operadores](http://reactivex.io/documentation/operators.html) que... operan sobre instancias de los observables. Esos operadores son funciones que reciben y retornan observables.

Podemos ver a los observables como _streams_, es decir una corriente de datos que circula por una tubería. Los operadores serán funciones que afecten al contenido o al caudal y que se pueden agregar o eliminar ordenadamente de la tubería. Una de esas operaciones se llama `tap`, un grifo.

> Nota: este operador fue anteriormente conocido como [`do`](http://reactivex.io/documentation/operators/do.html).

La operación `tap` se usa cuando se quiere actuar ante un cambio en el contenido o caudal pero sin cambiarlo. Para mi es adecuada porque lo que pretendo es auditar las llamadas y enterarme de los errores sin tocar absolutamente nada.

En base a todo lo anterior montaré una sentencias como estas dentro de la función `intercept`:

```typescript
const handledRequest = next.handle(req);
const successCallback = this.interceptResponse.bind(this);
const errorCallback = this.catchError.bind(this);
const interceptOperator = tap<HttpEvent<any>>(successCallback, errorCallback);
return handledRequest.pipe(interceptOperator);
```

Tómate tu tiempo para revisar cada línea. En primer lugar obtengo un puntero al _stream observable_ que es la petición en curso. Despues asigno dos funciones locales que actuarán como _callbacks_ para cuando lleguen datos o errores respectivamente. Preparo el operador `tap` de la librería observable asignádole ambos _callbacks_ y un tipo de retorno concreto en su genérico. Y por último mediante el método `pipe` engancho el operador a la tubería por la que circula el chorro. Respira y vuelve a leerlo.

> Admito que si es tu primer contacto con este mundo de los observables este codigo pueda resultar complejo. La buena noticia es que este código es una especie de patrón o _snippet_ que puedes reutiliazar para casi cualquier interceptor. Sólo habrá que cambiar el operador y el trabajo interno de sus _callbacks_.

Con esto tienes un sistema que envía a la consola información extra como la duración de las llamadas. Además inspecciona los errores en un único lugar. Esto puede incluso hacer innecesario que los componentes procesen errores.

Ya tenemos los datos almacenados en un servidor con el que nos ciumnicamos por _http_; aunque por ahora de forma anónima. Con el conocimiento actual de los observables, el _httpClient_ y los interceptores estamos a un paso de darle seguridad a las comunicaciones. Sigue esta serie para añadirle [vigilancia y seguridad en Angular](../categories/Tutorial/Angular/) mientras aprendes a programar con Angular5.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>
