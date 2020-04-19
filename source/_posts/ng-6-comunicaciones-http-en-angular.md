---
title: Comunicaciones http en Angular
permalink: comunicaciones-http-en-Angular
date: 2020-04-18 11:06:00
tags:
- Angular
- http
- RxJS
- Observables
- Tutorial
- Introducción
- Angular9
- Angular2
categories:
- [Tutorial, Angular]
thumbnail: /css/images/angular-6_http.png
---

![comunicaciones-http-en-Angular](/images/tutorial-angular-6_http.png)

Las comunicaciones _http_ son una pieza fundamental del desarrollo web, y en **Angular** siempre han sido potentes y fáciles. ¿Siempre?, bueno cuando apareció Angular 2 echábamos en falta algunas cosillas... y además la librería *RxJS* y sus *streams* son intimidantes para los novatos.

Pero en la versión Angular 9 **consumir un servicio REST** puede ser cosa de niños si aprendes a jugar con los _observables_ y los servicios de la librería `@angular/common/http`. Conseguirás realizar **comunicaciones http asíncronas en Angular**.

<!-- more -->

Partiendo de la aplicación tal como quedó en [Servicios inyectables en Angular](../servicios-inyectables-en-Angular/). Al finalizar tendrás una aplicación que almacena y recupera los datos consumiendo un servicio REST usando las tecnologías de Angular Http.

> Código asociado a este tutorial en _GitHub_: [AcademiaBinaria/angular-basic](https://github.com/AcademiaBinaria/angular-basic/)

# 1. El servicio HttpClient

Como demostración vamos a consumir un API pública con datos de [cotización de monedas](https://exchangeratesapi.io/). Crearé un módulo y un componente en el que visualizar las divisas y después las transformaremos para guardarlas en un servicio propio.

```shell
ng g m money --route money --module app-routing.module
```

## 1.1 Importación y declaración de servicios

La librería `@angular/common/http` trae el módulo `HttpClientModule` con el servicio inyectable `HttpClient`. Lo primero es importar dicho módulo.

```typescript
import { HttpClientModule } from '@angular/common/http';
@NgModule({
  declarations: [MoneyComponent],
  imports: [CommonModule, MoneyRoutingModule, HttpClientModule],
})
export class MoneyModule {}
```

En tu componente tienes que reclamar la dependencia al servicio para poder usarla. Atención a la importación pues hay más clases con el nombre `HttpClient`. Debe quedar algo así:

```typescript
import { HttpClient } from '@angular/common/http';
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'ab-money',
  templateUrl: './money.component.html',
  styles: []
})
export class MoneyComponent implements OnInit {
  constructor(private httpClient: HttpClient) {}

  ngOnInit() {}
}
```

A partir de este momento sólo queda invocar los métodos REST en la propiedad `this.http`.

## 1.2 Obtención de datos

Para cada verbo _http_ tenemos su método en el servicio `HttpClient`. Su primer parámetro será la *url* a la que invocar. Empecemos por el `get` que automáticamente solicita y devuelve objetos _JSON_ desde un API. Por ejemplo para obtener [las últimas cotizaciones de las principales divisas](https://api.exchangeratesapi.io/latest?symbols=USD,GBP,CHF,JPY) lo haremos así:


```typescript
export class RatesComponent implements OnInit {
  private urlapi
    = 'https://api.exchangeratesapi.io/latest';
  public currentEuroRates: any = null;

  constructor(private httpClient: HttpClient) {}

  ngOnInit() {
    this.getCurrentEuroRates();
  }

  private getCurrentEuroRates() {
    const currencies = 'USD,GBP,CHF,JPY';
    const url = `${this.urlapi}?symbols=${currencies}`;
    this.httpClient
      .get(url)
      .subscribe(apiData => (this.currentEuroRates = apiData));
  }
}
```

> El método _get_ retorna un objeto observable. Los observables _http_ han de consumirse mediante el método _subscribe_ para que realmente se lancen. Dicho método _subscribe_ admite hasta tres _callbacks_ para responder a tres sucesos posibles. Retorno de datos correcto, retorno de un error y señal de finalización. La sintaxis original ofrece tres argumentos opcionales para enviarles las funciones callback `susbcribe(data, err, end)`. En este ejemplo solo hemos usado el primero.

Otra sintaxis más reciente sustituye los tres argumentos funcionales por un un único objeto. La ventaja es que puedes usar clases con lógica común, reutilizarlos en distintas suscripciones, o simplemente tener el código un poco más organizadito.

```typescript
observable$.susbcribe({
  next: function(data){},
  error: function(err){},
  complete: function(){}
  })
```
La presentación en la vista sólo tiene que acceder a la propiedad dónde se hayan cargado las respuestas tratadas en el _callback_ de la suscripción.

```html
<h2> Currency Rates. </h2>
<h3> From Euro to the world </h3>
<pre>{{ currentEuroRates | json }}</pre>
```


## 1.3 Envío de datos

Supongamos que, una vez recibidas las cotizaciones, pretendemos transformarlas y almacenarlas en otro servicio. Por ejemplo un objeto para cada día y moneda. El envío en este caso será con el método _post_ al que se le pasará la ruta del _end point_ y el objeto _payload_ que se enviará al servidor.

Vamos a agregar una propiedad y un par de métodos al `rates-component.ts`. La idea es obtener un array de cotizaciones aa partir del objeto previo, y guardarla una por una.

```typescript
this.httpClient
      .post(url, payloadObject)
      .subscribe();
```
> Atención a los métodos `subscribe()`. Aunque vayan vacíos son imprescindibles para que se ejecute la llamada. Esto puede resultar contra intuitivo, pero la verdad es que los observables Http de Angular sólo trabajan si hay alguien mirando.


## 1.4 Actualización de datos

Un par de seudo ejemplos más para acabar de entender la mecánica básica de `HttpClient`. Podemos fijar el tipo de datos esperado en cualquier llamada. De hecho es recomendable que tengas un interfaz para cada respuesta esperada.

```typescript
public update() {
  this.httpClient.delete(itemResourceUrl, newValue).subscribe();
}
public delete() {
  this.httpClient.delete(itemResourceUrl).subscribe();
}
```

Y hasta aquí lo básico de comunicaciones *http*. ¿Fácil verdad?. Pero la vida real raramente es tan sencilla. Si quieres enfrentarte a algo más duro debes prepararte y dominar los observables *RxJS*.

Lo que viene a partir de ahora requerirá tiempo y concentración. Si continúas adelante esto ya no será *your grandpa´s http anymore*.

 # 2 Observables

Las **comunicaciones** entre navegadores y servidores son varios órdenes de magnitud más lentas que las operaciones en memoria. Por tanto deben realizarse de manera asíncrona para garantizar una buena experiencia al usuario.

Esta experiencia no siempre fue tan buena para el programador. Sobre todo con las primeras comunicaciones _AJAX_ basadas en el paso de funciones _callback_. La aparición de las _promises_ mejoró la claridad del código, y ahora con los _Observables_ tenemos además una gran potencia para manipular la **información asíncrona**.

> El patrón `Observable` fue implementado por Microsoft en la librería [_Reactive Extensions_](http://reactivex.io/intro.html) más conocida como `RxJs`. El equipo de Angular decidió utilizarla para el desarrollo de las comunicaciones asíncronas. Esta extensa librería puede resultar intimidante en un primer vistazo. Pero con muy poco conocimiento puedes programar casi todas las funcionalidades que se te ocurran.

Lo primero es importar el código, de forma similar a cualquier otra clase o función. Por ejemplo para empezar basta con `import { Observable } from "rxjs/Observable";`. Tendremos la clase usada por angular para observar el respuesta *http*.

Pero esta es una clase genérica donde sus instancias admiten la manipulación interna de tipos más o menos concretos. Por eso ves en el ejemplo que algunas funciones retornan el tipo esperado `: Observable<MyClass>`, o si no saben que tipo esperar se conforman con `: Observable<any>`.

En cualquier caso, **toda operación asíncrona retornará una instancia observable** a la cual habrá que subscribirse para recibir los datos o los errores, cuando termine.

Aunque a veces no se verá el _subscribe_...

Ya tenemos el programa comunicado por _http_ con un servidor; aunque por ahora de forma anónima y sin ninguna seguridad. Necesitamos aumentar el conocimiento actual de los observables, del _httpClient_ con los interceptores y ya estaremos cerca de resolverlo. Sigue esta serie para añadirle [vigilancia y seguridad en Angular](../vigilancia-y-seguridad-en-Angular/) mientras aprendes a programar con Angular. Todos esos detalles se tratan en el [curso básico online](https://www.trainingit.es/curso-angular-basico/?promo=angular.builders) que imparto con TrainingIT o a medida para tu empresa.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>
