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

Las comunicaciones _http_ son una pieza fundamental del desarrollo web, y en **Angular** siempre han sido fáciles y potentes. ¿Siempre?, bueno cuando apareció Angular 2 echábamos en falta algunas cosillas. Pero con la versión actual **consumir un servicio REST** vuelve a ser cosa de niños.

Claro que para ello tendremos que jugar con los _observables_ y los servicios de la librería `@angular/common/http` con los que realizar **comunicaciones asíncronas en Angular**.

<!-- more -->

Partiendo de la aplicación tal cómo quedó en [Servicios inyectables en Angular](../servicios-inyectables-en-Angular/). Al finalizar tendrás una aplicación que almacena y recupera los datos consumiendo un servicio REST.

> Código asociado a este artículo en _GitHub_: [AcademiaBinaria/angular5/6-http](https://github.com/AcademiaBinaria/angular5/tree/master/6-http/cash-flow)
> El servicio REST se encuentra en _GitHub_: [AcademiaBinaria/ApiBase](https://github.com/AcademiaBinaria/ApiBase)

# 1. El servicio HttpClient

La librería `@angular/common/http` trae el módulo `HttpClientModule` con el servicio inyectable `HttpClient` que debes declarar como dependencia en tus propios constructores.

En el fichero `operations.service.ts` tienes el código que reclama la dependencia y la configura con una ruta base obtenida de la configuración de `environment`. Queda algo así:

```typescript
export class OperationsService {
  private url = environment.apiUrl + "pub/items/";

  constructor(private http: HttpClient) {}
}
```

A partir de este momento sólo queda invocar los métodos REST en la propiedad `this.http`.

# 2 Métodos REST

Para cada verbo _http_ tenemos su método en el servicio `HttpClient`. Su primer parámetro será la url a la que invocar. Los métodos de envío reciben la carga en el segundo argumento, y la envían automáticamente como objetos _JSON_.

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

> Cada método de negocio, configura la llamada de infraestructura; parece poca cosa. Podría ser un buen sitio para validar la información antes de ser enviada, o quizás agrupar varias llamadas de red para una misma operación de negocio. El _dolar_ al final del nombre es un convenio para las funciones que devuelven observables.

# 3 Observables

Las **comunicaciones** entre navegadores y servidores son varios órdenes de magnitud más lentas que las operaciones en memoria. Por tanto deben realizarse de manera asíncrona para garantizar una buena experiencia al usuario.

Esta experiencia no siempre fue tan buena para el programador. Sobre todo con las primeras comunicaciones _AJAX_ basadas en el paso de funciones _callback_. La aparición de las _promises_ mejoró la claridad del código, y ahora con los _Observables_ tenemos además una gran potencia para manipular la **información asíncrona**.

> El patrón `Observable` fue implementado por Microsoft en la librería _Reactive Extensions_ más conocida como `RxJs`. El equipo de Angular decidió utilizarla para el desarrollo de las comunicaciones asíncronas. Esta extensa librería puede resultar intimidante en un primer vistazo. Pero con muy poco conocimiento puedes programar casi todas las funcionalidades que se te ocurran.

Lo primero es importar el código, esto se hace forma similar a cualquier otra clase o función. Para empezar basta con `import { Observable } from "rxjs/Observable";`.

Esta es una clase genérica donde sus instancias admiten la manipulación interna de tipos más o menos concretos. Por eso ves en el ejemplo que algunas funciones retornan `: Observable<Operation>`, o si no saben que tipo esperar se conforman con `: Observable<any>`.

En cualquier caso, **toda operación asíncrona retornará una instancia observable** a la cual habrá que subscribirse para recibir los datos o los errores, cuando termine. Por limpieza eso debes hacerlo en los componentes que consuman el servicio, no en el propio servicio.

## 3.1 Lectura

En `operations.component` están las llamadas y las subscripciones necesarias. Por ejemplo el método `refreshData()` realiza llamadas y se suscribe para conocer los resultados.

```typescript
private refreshData() {
 this.operationsService
   .getOperationsList$()
   .subscribe(
     data => (this.operations = data),
     err => this.catchError(err)
   );
}
```

El método `subscribe` recibe hasta tres argumentos (ok, error, fin) donde colocar funciones receptoras para cada tipo de evento que ocurra. Sólo el primero es obligatorio, y es en el que recibes la información directamente desempaquetada. En el segundo, normalmente pondrás lógica para responder ante códigos de error devueltos por el servidor.

> Recordatorio para novatos: Es importante comprender la naturaleza asíncrona de estas operaciones. El código de las funciones subscritas se ejecutará en el futuro, no de una manera secuencial.

## 3.2 Escritura

Si lo que quieres es enviar objetos a un servidor, por ejemplo mediante el verbo _POST_, sólo tienes que pasarle la _payload_ al método de negocio y suscribirte a la respuesta.

```typescript
public saveOperation(operation: Operation) {
 this.operationsService
   .saveOperation$(operation)
   .subscribe(data => this.refreshData());
}
```

> De nuevo, fíjate como refrescamos los datos una vez recibida la respuesta. Hacerlo antes podría dar lugar a respuestas incongruentes. En este caso no es la respuesta en sí lo que interesa, sino el hecho de haya terminado bien.

Ya tenemos los datos almacenados en un servidor, aunque por ahora de forma anónima. Sigue esta serie para añadirle [vigilancia y seguridad en Angular](../categories/Tutorial/Angular/) mientras aprendes a programar con Angular5.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>
