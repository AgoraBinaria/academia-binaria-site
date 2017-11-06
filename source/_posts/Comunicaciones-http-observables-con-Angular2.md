---
title: Comunicaciones http observables con Angular2
tags:  
- Angular2
categories:
- Introducción 
permalink: comunicaciones-http-observables-con-angular2
id: 17
updated: '2017-02-16 17:44:21'
date: 2016-06-16 11:47:28
thumbnail: /css/images/angular.jpg
---

En AngularJS, y en otros frameworks del lado cliente, la idea de fue simple desde el principio: dame **plantillas estáticas y datos dinámicos**. Las comunicaciones *http* son las arterias vitales de transporte de esos datos dinámicos. Y en Angular 2 se han revisado por completo, llevando las comunicaciones asíncronas al siguiente nivel.

<!-- more -->

### La librería http y otras...
Como cabe esperar Angular 2 dispone de su propio módulo de comunicaciones. En la librería `@angular/http` encontramos el nuevo servicio `http` que es el cliente usado para enviar y recibir datos.

Lo realmente novedoso viene de parte del proyecto [Reactive Extensions o RxJS](http://reactivex.io/rxjs/). En su librería `rxjs/Observable` exporta la clase `Observable`. Esta clase implementa el **patrón observador aplicado a streams** de datos. El equipo de Angular ha decidido adoptarlo para procesar los *streams* de entrada y salida de datos *http*.

Un servicio típico que necesite comunicaciones con el servidor importará al menos estos objetos.

```typescript
// Importar objetos de la librería http
import { Http, Response, RequestOptions, Headers } from '@angular/http';
// Importar la clase Observable desde la librería rxjs
import { Observable }     from 'rxjs/Observable';
```

Los servicios importados han de ser **registrados como *providers* para poder ser inyectados como dependencias** antes de ser consumidos. Ya que se trata de servicios de amplio uso en cualquier aplicación, se recomienda registrarlos en módulos de alto nivel. De esa forma pueden ser reutilizados como *singletons*.

```typescript
// importar la constante con los proveedores de http
import { HttpModule, Http } from '@angular/http';

@NgModule({
  declarations: [ ],
  imports: [
    HttpModule, // El módulo con todo lo necesario
  ],
  providers:[
    Http, // El servicio proveedor
  ]
});
```

A partir de ese momento cualquier componente o servicio puede reclamar su inyección en el constructor, y lo podrá usar en sus propios métodos.
```typescript
  /**
  * Constructor que reclama dependencias inyectables
  * Http se encuentra por haberse registrado en este módulo o en uno superior
  **/
  constructor(private http: Http) {
     // en el constructor no debe contener lógica extra
     // su función es únicamente recibir las dependencias
  }
```

### Observables en lugar de promesas
La naturaleza asíncrona de las comunicaciones entre maquinas se implementó mediante *callbacks* en JavaScript. Esta forma de programar degenera en código difícil de mantener. Con el tiempo el **patrón promesa** se impuso, y en AngularJS 1.x es la manera recomendada de programar.

Pero las promesas también tiene sus limitaciones, y ahí aparecen los **observables**. Este patrón requerirá un artículo para el sólo, pero como adelanto se resume en lo siguiente:
> Tratar todo tipo de información como un stream observable de entrada y de salida, al cual se le pueden agregar operaciones que procesan los flujos de datos.

Esto encaja muy bien con las comunicaciones *http* asíncronas y es la razón de su adopción en Angular 2. Un método de servicio par **leer datos REST** se parecerá a este *snippet*:

```typescript
// las llamadas devuelven observables
leerDatos(): Observable<Response> {
  // Se declara cómo va a ser la llamada 
  // ocultando los pormenores a los consumidores   
  return this.http
    .get(`${this.urlBase}/recurso`);
  // En este momento aún no se efectuó la llamada
}
```

Para enviar información via *post* o *put* usaremos una estrategia similar, pero con unos requisitos específicos. En Angular 2 se requiere una configuración previa de las llamadas de escritura. Es engorroso pero fácilmente automatizable. Una típica operación de **escritura REST** sería algo así:

```typescript
  escribirDatos(unDato): Observable<Response> {
    // Los envíos de información deben configurarse a mano
    // esto es fácilmente generalizable y reutilizable
    let body = JSON.stringify(unDato);
    let headers = new Headers({ 'Content-Type': 'application/json' });
    let options = new RequestOptions({ headers: headers });
    // declarar la llamada y retornar el observable
    // las variables de configuración y los datos, van como parámetros
    if (unDato._id) {
      return this.http
        .put(`${this.urlBase}/recurso/${unDato._id}`, body, options);
    } else {
      return this.http
        .post(`${this.urlBase}/recurso`, body, options);
    }
  }
```


Por supuesto esto es lo que se programa a bajo nivel, en los **servicios de comunicaciones**. En Angular2 se mantiene la recomendación de bajar a servicios la responsabilidad de las comunicaciones al tiempo que dejamos los componentes lo más ligeros posible. 

Por encima de los servicios de comunicaciones habrá otros servicios de lógica o directamente componentes. En cualquier caso, los consumidores llamarán a métodos que siempre les devolverán objetos observables. Las clases de alto nivel se **suscribirán a esos observables** y procesarán la respuesta recibida... cuando esta esté disponible.

```typescript
  // La carga de datos se hace al iniciarse el componente
  // este es el lugar donde programar lógica de inicio
  // nunca en el constructor
  ngOnInit() {
    // en el momento de la suscripción es cuando se dispara la llamada
    this.datosService
      .leerDatos()
      .subscribe(res => {
        this.datos = res.json();
      });
    // Sería similar en procesos de escritura
  }
```
> La clase `http` no realiza la llamada hasta que algún consumidor se suscriba a la respuesta. Mientras tanto queda como una definición congelada.

El método `.subscribe()` recibe como argumento un puntero a la respuesta *http*. Los datos se encuentran en formato *JSON* y hay que reclamarlos mediante el método `.json()`

### Extensiones en lugar de interceptores
Una de las características destacables de los servicios `$http` de AngularJS 1.x era la posibilidad de usar *interceptores*. Estos eran **funciones que se incrustaban durante el envío o recepción** de las comunicaciones.

Un uso habitual era emplearlos para agregar **cabeceras de seguridad o controlar errores** de comunicación de manera centralizada. Ahora la forma recomendable de hacer eso mismo en Angular2 es mejorar el servicio http extendiéndolo. El servicio derivado resultante implementará los interceptores en funciones llamadas durante la comunicación.

Esta es una posible implementación con funcionalidad suficiente para facilitar los envíos, detectar errores y enviar cabeceras de seguridad.
```typescript
@Injectable()
/**
 * Extensión personalizada de la clase HTTP
 * Permite la configuración de todas las peticiones
 * Captura los envíos y respuestas
 * */
export class HttpService extends Http {
  /** Las direcciones base deberían venir de la configuración del environment*/
  public apiProxyUrl = 'http://localhost:4030/api/';
  private authorization = '';

  constructor(
    backend: XHRBackend,
    defaultOptions: RequestOptions,
    private router: Router,
    private userStore: SessionStoreService
  ) {
    super(backend, defaultOptions);
    this.subscribeToToken();
  }

  /**
   * Reescribe el método de la clase base, ejecutando acciones para cada petición
   * La peticiíón en curso puede llegar como una ruta o una clase request
   * Si viene sólo la cadena, debería traer las opciones aparte
   * */
  request(request: string | Request, options: RequestOptionsArgs = { headers: new Headers() }): Observable<Response> {
    this.configureRequest(request, options);
    return this.interceptResponse(request, options);
  }

  private subscribeToToken() {
    // suponemos un servicio que nos avisa de la recepción de tokens
    this.userStore
      .getDataObservable()
      .subscribe((data: Session) => this.authorization = 'Bearer ' + data.token);
  }

  private configureRequest(request: string | Request, options: RequestOptionsArgs) {
    // Adapta la ruta y asigna cabeceras
    if (typeof request === 'string') {
      request = this.getProxyUrl(request);
      this.setHeaders(options);
    } else {
      request['url'] = this.getProxyUrl(request['url']);
      this.setHeaders(request);
    }
  }

  private interceptResponse(request: string | Request, options: RequestOptionsArgs) : Observable<Response> {
    const observableRequest = super
      .request(request, options)
      .catch(this.onCatch())
      .finally(this.onFinally());
    return observableRequest;
  }

  /**
   * Transforma la url para llamar a trave´s de un proxy 
   * Útil en caso de problemas con el CORS
   */
  private getProxyUrl(currentUrl) {
    if (!currentUrl.includes('/assets/')) {
      return this.apiProxyUrl + currentUrl;
    } else {
      return currentUrl;
    }
  }

  /**
   * Interceptor para componer las cabeceras en cada petición
   * */
  private setHeaders(objectToSetHeadersTo: Request | RequestOptionsArgs) {
    const headers = objectToSetHeadersTo.headers;
    headers.set('Content-Type', 'application/json');
    headers.set('Authorization', this.authorization);
  }

  /**
   * Interceptor para captura genérica de errores http
   * */
  private onCatch() {
    return (res: Response) => {
      // Security errors
      if (res.status === 401 || res.status === 403) {
        // redirigir al usuario para pedir credenciales
        this.router.navigate(['user/login']);
      }
      // To Do: Gestión común de otros errores...
      return Observable.throw(res);
    };
  }

  private onFinally() {
    return () => console.log('Fin');
  }

}
```


Podemos usarlo de manera transparente en el resto de la aplicación. Sólo se necesita ajustar el registro de proveedores para dirigir al inyector de dependencias hacia esta nueva clase.
```typescript
   providers: [
    {
      provide: Http, // reemplaza el servicio del framework
      useClass: HttpService // con la clase personalizada que lo extiende
    }
  ]
```

>Este es uno de los usos más potentes de la inversión de control en Angular. Es el programador integrador el que decide el servicio a usar. El programador funcional declara la dependencia en los constructores de sus componentes y servicios, pero no decide la instancia concreta que recibirá.


Angular 2 ha trastocado todo lo que hacíamos en las versiones 1.x, pero sólo en la forma. El fondo sigue siendo el mismo: **el navegador envía y recibe datos en formato JSON que mezcla con plantillas estáticas** para  crear HTML en la máquina local y relajar al servidor. 


Por cierto, por ahora ni rastro del viejo `$resource`. No tardará en aparecer una versión basada en *streams observables*. Sígueme en [mi cuenta de twitter](https://twitter.com/albertobasalo) o suscríbete a la [revista mensual de Academia Binaria](http://academia-binaria.us4.list-manage.com/subscribe?u=c8ad2d2e7d02c26e32ce4cded&amp;id=b67e4d2339) y en cuanto me entere serás el primero en ser informado.

*Keep coding, keep learning.*