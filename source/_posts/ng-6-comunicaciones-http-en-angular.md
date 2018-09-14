---
title: Comunicaciones http en Angular
permalink: comunicaciones-http-en-Angular
date: 2018-09-14 11:06:00
tags:  
- Angular
- http
- Observables
- Tutorial
- Introducción
- Angular6
- Angular5
- Angular2
categories:
- [Tutorial, Angular] 
thumbnail: /css/images/angular-6_http.png
---

![Tutorial Angular5 6-http](/images/tutorial-angular-6_http.png)

Las comunicaciones _http_ son una pieza fundamental del desarrollo web, y en **Angular** siempre han sido fáciles y potentes. ¿Siempre?, bueno cuando apareció Angular 2 echábamos en falta algunas cosillas, y la librería *RxJs* y sus *streams* son intimidantes. 

Pero con la versión 6 **consumir un servicio REST** puede ser cosa de niños si aprendes a jugar con los _observables_ y los servicios de la librería `@angular/common/http`. Conseguirás realizar **comunicaciones asíncronas en Angular 6**.

<!-- more -->

Partiendo de la aplicación tal cómo quedó en [Servicios inyectables en Angular](../servicios-inyectables-en-Angular/). Al finalizar tendrás una aplicación que almacena y recupera los datos consumiendo un servicio REST.

> Código asociado a este artículo en _GitHub_: [AcademiaBinaria/AutoBot/6-http](https://github.com/AcademiaBinaria/autobot/tree/6-http)
> > Tienes una versión desplegada operativa para probar [AutoBot](https://academiabinaria.github.io/autobot/) 

# 1. El servicio HttpClient

La librería `@angular/common/http` trae el módulo `HttpClientModule` con el servicio inyectable `HttpClient` que debes declarar como dependencia en tus propios constructores.

En tu código tienes que reclamar la dependencia para poder usarla. Queda algo así:

```typescript
export class CarsService {
  constructor(private http: HttpClient) {}
}
```

A partir de este momento sólo queda invocar los métodos REST en la propiedad `this.http`.

## 1.1 Métodos REST

Para cada verbo _http_ tenemos su método en el servicio `HttpClient`. Su primer parámetro será la *url* a la que invocar. Los métodos de envío reciben la carga en el segundo argumento, y la envían automáticamente como objetos _JSON_.

Veamos un ejemplo sencillo para montar un *CRUD* de cualquier cosa, coches por ejemplo. He usado una notación similar al *SQL* para ilustrar lo que se espera de los comandos.

```typescript
 public selectCars$(): Observable<Car[]> {
   return this.http.get<Car[]>('https://autobot.com/api/cars/');
 }
 public selectCar$(carId: string): Observable<Car> {
   return this.http.get<Car>('https://autobot.com/api/cars/' + carId);
 }
 public insertCar$(car: Car): Observable<car> {
   return this.http.post<Car>('https://autobot.com/api/cars/', car);
 }
public updateCar$(car: Car): Observable<car> {
   return this.http.put<Car>('https://autobot.com/api/cars/' + car._id, car);
 }
 public deleteCar$(operation: Operation): Observable<any> {
   return this.http.delete('https://autobot.com/api/cars/' + car._id);
 }
```

> Cada método de negocio, configura la llamada de infraestructura; parece poca cosa. Podría ser un buen sitio para validar la información antes de ser enviada, o quizás agrupar varias llamadas de red para una misma operación de negocio. El _dolar_ al final del nombre es un convenio para las funciones que devuelven observables.

## 1.2 Subscribe

El consumo de este servicio en su versión más básica sería algo así:

```typescript
export class CarsComponent {
  constructor(private cars: CarsService) {}

  public ngOnInit(){
    this.cars.selectCars$()
      .susbcribe(
        cars => console.log('I have ' + cars.lenght + ' cars.'),
        err => console.error('There was an error: ' + err.message)
      )
  }
}
```

El método de nuestro servicio nos devuelve un observable de los datos de la respuesta *http*. Es un *stream* de un sólo suceso pero al que alguien debe subscribirse. En la susbcripción asignamos funciones *callback*. La primera se ejecutará al recibir los datos. La segunda en caso de que haya un error.

Y hasta aquí lo básico de comunicaciones *http*. ¿Fácil verdad?. Pero la vida real raramente es tan sencilla. Si quieres enfrentarte a algo más duro debes prepararte y dominar los observables *RxJs*. 

Lo que viene a partir de ahora requerirá tiempo y concentración. Si continúas adelante esto ya no será *your grandpa´s http anymore*.

# 2 Observables

Las **comunicaciones** entre navegadores y servidores son varios órdenes de magnitud más lentas que las operaciones en memoria. Por tanto deben realizarse de manera asíncrona para garantizar una buena experiencia al usuario.

Esta experiencia no siempre fue tan buena para el programador. Sobre todo con las primeras comunicaciones _AJAX_ basadas en el paso de funciones _callback_. La aparición de las _promises_ mejoró la claridad del código, y ahora con los _Observables_ tenemos además una gran potencia para manipular la **información asíncrona**.

> El patrón `Observable` fue implementado por Microsoft en la librería [_Reactive Extensions_](http://reactivex.io/intro.html) más conocida como `RxJs`. El equipo de Angular decidió utilizarla para el desarrollo de las comunicaciones asíncronas. Esta extensa librería puede resultar intimidante en un primer vistazo. Pero con muy poco conocimiento puedes programar casi todas las funcionalidades que se te ocurran.

Lo primero es importar el código, esto se hace forma similar a cualquier otra clase o función. Para empezar basta con `import { Observable } from "rxjs/Observable";`.

Esta es una clase genérica donde sus instancias admiten la manipulación interna de tipos más o menos concretos. Por eso ves en el ejemplo que algunas funciones retornan `: Observable<Car>`, o si no saben que tipo esperar se conforman con `: Observable<any>`.

En cualquier caso, **toda operación asíncrona retornará una instancia observable** a la cual habrá que subscribirse para recibir los datos o los errores, cuando termine. Por limpieza eso debes hacerlo en los componentes que consuman el servicio, no en el propio servicio.

## 2.1 Pipe

En `HomeComponent` tenemos un ejemplo sencillo para empezar con el consumo de observables. En una llamda al servicio obtemos como resultado un Observable. Pero mira el código relevante y verás que las cosas cambian:

```typescript
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';
export class HomeComponent implements OnInit {
  public carLinks$: Observable<Link[]>;

  constructor(private cars: CarsService) {}

  public ngOnInit() {
    this.carLinks$ = 
      this.cars.getCars$()
      .pipe(
        map(
          (cars: Car[]): Link[] => cars.map(this.getLinkFromCar)
        )
      );
  }
  private getLinkFromCar(car: Car): Link {
    return {
      caption: car.link.caption,
      routerLink: '/car/' + car.link.routerLink,
      value: formatNumber(car.cost, 'en-US') + ' EUR'
    };
  }
}
```

## 2.1.1 El operador map

Los datos devueltos raramente vienen en el formato preciso para usar en la vista. Con frecuencia hay que transformarlos al vuelo en cuanto se reciben. Recordemos que ***HttpClient* no devuelve los datos tal cual sino un *stream* de estados** de dichos datos. La manipulación será sobre el *stream* no direactamente sobre los datos y para manipular un torrente hay que encauzarlo en tuberías.

Aquí es donde aparece el método `.pipe(operator1, operator2...)` aplicado a un observable, en luagar o antes de `.susbcribe(okCallback, errCallback)`. Este método entuba una serie de operadores predefinidos que manipulan el chorro de estados observados.

El operador más utilizado es `map(sourceStream => targetStream)`. Este operador recibe una función *callBack* que será invocada ante cada dato recibido. Esa función tienen que retornar un valor para sustituir al actual y así transformar el contenido del chorro.

> En este caso partimos de un *array* de objetos de tipo `Car[]` y lo transformamos en otro array de tipo `Link[]`. Para ello, forzando un poco el ejercicio, he usado la función `.map(sourceElement => targetElement)` de los *arrays Javascript*. El conflicto es intencionado por mi parte para mostrar la coincidencia en nombre y propósito. Pero son cosas muy distintas pues el operador `map` ha de importarse de `rxjs/operators` y aplicarse a un observable dentro de su método `.pipe()`. Nada que ver con la sencilla función `array.map(callback)`.

## 2.1.2 El pipe async

Siguiendo con las conicidencias en nombres que suelen causar confusión está el *pipe* `async`. En este caso es tecnología puramente Angular. En las vistas usamos `|` para trasnformar la represntación de un dato mediante una función. Ene ste caso, la función recibirá un onservable y retornará su contenido. 

```html
  <app-menu-list caption="Cars in your garage:"
    [links]="carLinks$ | async">
  </app-menu-list>
```

Este es el modo recomendado de consumo de observables en la vista. Hacerlo mediante el *pipe* `observable<any> | async` aporta múltples venjas que se irán viendo en este tutorial. Este *pipe* de Angular es una función que se subscribe al torrente recibido y devuelve sus estados concretos. 

Se podría, pero no se recomienda, haber resuelto el tema de la siguiente manera:

```typescript
@Component({
  selector: 'app-home',
  template: `
  <app-menu-list caption="Cars in your garage:"
    [links]="carLinks">
  </app-menu-list>
  `,
  styles: [],
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class HomeComponent implements OnInit {
  public carLinks: Link[];
  constructor(private cars: CarsService) {}

  public ngOnInit() {
    this.carLinks$ = this.cars.getCars$()
      .pipe(
        map(
          (cars: Car[]): Link[] => cars.map(this.getLinkFromCar)
        )
      )
      .subscribe(
        (links: Link[]): void => this.links= links; // Not recomended
      );
  }
  private getLinkFromCar(car: Car): Link {
    return {
      caption: car.link.caption,
      routerLink: '/car/' + car.link.routerLink,
      value: formatNumber(car.cost, 'en-US') + ' EUR'
    };
  }
}
```

> Recordatorio para novatos asíncronos: Es importante comprender la naturaleza asíncrona de estas operaciones. El código de las funciones *callbacks* subscritas se ejecutará en el futuro, no de una manera secuencial.

## 2.2 Switch Map 

Hemos visto como recuperar información asícrona y recibirla con un observable. Hemos aprendido a transformar los datos recibidos. Pero, ¿qué ocurre si lo que necesito para la transformación es otra llamada síncrona?.

El operador `map()` trabaja con el contenido de un *stream*. Pero si queremos derivar el curso hacia otro arroyo necesitamos un operador más poderoso: el `switchMap()`.

Esta es una situación habitual cuando llamamos a un servicio y con el resultado tenemos que llamar a otro. En realidad lo que queremos es el resultado del segundo observable, no del primero. Para ello cambiamos al vuelo uno por otro; de ahí el *switch*.

En el `car.component.ts` tienes un ejemplo completo y complejo del entubado encadenado de operadores. Además del mencionado `switchMap` te presento también al sencillo `tap()`. Este operador se usa cuando queremos ejecutar instrucciones que no manipulen los datos del *stream*. Pero que se efectúen cuando los datos lleguen. Es decir, *tap* usa el dato pero no lo transforma.

```typescript
public ngOnInit() {
    this.subscription = this.route.params
      .pipe(
        map((params: Params): string => params['carId']),
        switchMap((carId: string): Observable<Car> => this.cars.getCarByLinkId$(carId)),
        tap(this.onCarGotten),
        switchMap((car: Car): Observable<Car> => this.travels.getCarTravel$(car)),
        switchMap((car: Car): Observable<number> => interval(environment.refreshInterval))
      )
      .subscribe(this.timeGoesBy);
  }
```
La narrativa de lo que sucede en esta fucnión es la siguiente:

- 1 - Partimos de  `this.route.params` que es un *stream* observable de los estados de los parametros de la ruta activa que devuelve `Observable<Params>`.

- 2 - Extraemos el parámetro `carId`, que es un `string`.
- 3 - Realizamos una llamada asincrona para obtener el coche con es identificador. El resultado es un nuevo *stream* que retorna `Observable<Car>`.
- 4 - Almacenamos el coche e inicializamos indicadores mediante `tap`. Sin cambiar nada en el curso del arroyo.
- 5 - Para continuar el viaje hay que llamar de nuevo al servidor y de forma asíncrona completar o rellenar datos del coche. De nuevo una derivación en la corriente, auqne el tipo siga siendo el mismo: `Observable<Car>` .
- 6 - Al tener los datos ya completos queremos ceder el control al usuario y refrescar los indicadores según su manejo. La repetición de funciones a intervalos encja en la filosofía *Observable* y se incorpora en *RxJs* mediante la función `interval(period)` que retorna una secuencia de números a cada intervalo programado.
- 7 - Por último podemos ejecutar una función *callback* ante cada suceso usando la función `.subscribe(callback)`. 


Reconozco que en un primer vistazo este código pueda resultar complejo. Tómate tu tiempo. Fíjate en los tipos de entrada y salida de cada función. Revísa la aplicación [Autobot en el repositorio](https://github.com/AcademiaBinaria/autobot/tree/6-http). Especialmente en `cars.service.ts` y `travels.service.ts` encontrarás muchos más ejemplos del trabajo con la librería *RxJs* y la manipulación de *streams de eventos observables*.

Ya tenemos un servidor con el que nos comunicamos por _http_; aunque por ahora de forma anónima. Con el conocimiento actual de los observables y el _httpClient_ estamos a un paso de darle seguridad a las comunicaciones. Sigue esta serie para añadirle [vigilancia y seguridad en Angular](../vigilancia-y-seguridad-en-Angular/) mientras aprendes a programar con Angular6.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>
