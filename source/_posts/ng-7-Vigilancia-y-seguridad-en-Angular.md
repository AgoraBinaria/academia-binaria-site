---
title: Vigilancia y seguridad en Angular
permalink: vigilancia-y-seguridad-en-Angular
date: 2018-09-20 11:49:27
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
thumbnail: /css/images/angular-7_watch.png
---

![vigilancia-y-seguridad-en-Angular](/images/tutorial-angular-7_watch.png)

La seguridad de los datos es una responsabilidad compartida entre el servidor y el cliente. En **Angular 6** usaremos los _interceptores_ para detectar intrusos y enviar credenciales. Veremos como los _guards_ nos permiten controlar la navegación interna y los _resolvers_ nos aseguran los datos por adelantado.

 La **identificación de usuarios y el control** de acceso y navegación es parte del trabajo de un desarrollador front-end. Veremos nuevos usos de los _observables_ y los servicios de la librerías `@angular/common/http` y `@angular\router` con los que tratar  **comunicaciones seguras y fluidas en Angular**.

<!-- more -->

Partiendo de la aplicación tal cómo quedó en [Comunicaciones http en Angular](../comunicaciones-http-en-Angular/). Al finalizar tendrás una aplicación que identifica usuarios y se responsabiliza de almacenar y comunicar el _token_ de seguridad de un servicio REST y mejora la usabilidad controlando la entrada y la salida en las páginas.

> Código asociado a este artículo en _GitHub_: [AcademiaBinaria/AutoBot/7-watch](https://github.com/AcademiaBinaria/autobot/tree/7-watch)
> > Tienes una versión desplegada operativa para probar [AutoBot](https://academiabinaria.github.io/autobot/) 


# 1 Resolver

Cuando las una ruta se activa, el router de angular carga un componente en la vista. Bien, pero normalmente necesitará unos datos para ser mostrados. Y casi siempre serán datos de obrtención asíncrona. Lo cual nos lleva a tener que esperar antes de mostrar y mostrar vacíos, o cargando... En cualquier caso tendremos algún que otro *if* comprobando si los datos están listos o no.

Una posible solución vienen de la mano de los *resolvers*. Son servicios que implementan la interfaz `Resolve<any>` y que se ejecutan (resuelven) antes de cargar el componente de la ruta activa. Cuando termina avisa al router y este pone a disposición del componente los datos ya cargados, liberándolo de la lógica de carga, espera y comprobación. A cambio, la transición hacia la ruta no es tan fluida y agradece una buena animación.

Veamos por ejemplo la ruta raíz de Autobot que necesita la lista de coches para empezar. Esta solución invoucra tres ficheros.

## 1.1 Interfaz Resolve

En el fichero `cars-resolver.service.ts` partimos de un servicio inyectable local y le hacemos implementar la interfaz `Resolve<Cars[]>`:

```typescript
@Injectable()
export class CarsResolverService implements Resolve<Car[]> {
  constructor(private cars: CarsService) {}
  public resolve = (route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<Car[]> => this.cars.getCars$();
}
```
El método devuelve un observable al que se subscribe el router.

## 1.2 Routes

En el fichero de configuración de rutas `home-routing.module.ts` nos encontramos con el uso del `CarsResolverService`

```typescript
const routes: Routes = [{
  path: '',
  component: HomeComponent,
  resolve: {
    cars: CarsResolverService
  }
}];
```

Lo que hace el Angular Router es subscribirse al método `resolve` y almacenar su respuesta en una variable, en este caso llamada `cars`. Dicha variable será consumida de forma síncrona por el componente.

## 1.3 Snapshot data

El `HomeComponent` no interactúa directamente con el resolver y por tanto no reclama el servicio en sus dependencias. Símplemente trabaja con el router.

```typescript
  public cars: Car[];
  constructor(private route: ActivatedRoute) {}
  public ngOnInit() {
    this.cars = this.route.snapshot.data.cars;
  }
```

No hay necesidad de subscripción ni de control extra en la vista sobre la propiedad `cars`. Este patrón es recomendable cuando no queremos mostrar una vista a medio cargar, o cuando la lógica de carga asíncrona complica demasiado el componente.

# 2 Interceptor

Hasta ahora todas las comunicacione con nuestro servidor ha sido anónimas. Los datos eran públicos y no se exigía nada para poder leer o guardar información. Pero no siempre es así y los procesos de envío y recepción de datos del servidor suelen ser nominales y por tanto requieren de una identificación del usuario.

La seguridad de las comunicaciones con un servicio REST se resuelve habitualmente mediante una **credencial generada por el servidor llamada _token_**. Un usuario registrado en el sistema puede hacer _log in_ enviando una vez su identificador y contraseña. Si todo va bien, a cambio el servidor le enviará un _token_ que deberá usar en las siguientes llamadas. Con esto el servidor será capaz de autentificar las llamadas y responder adecuadamente.

La detección de fallos y el envío de credenciales es algo que haremos a casi todas las llamadas http. Parece razonable disponer de un mecanísmo único que se aplique a todas las llamdas sin necesidad de ir una por una. En Angular, eso se consigue con los interceptores.

## 2.1 Interfaz HttpInterceptor

El módulo http de Angular nos ofrexe una interfaz para implementar en servicios estándar y convertirlos en interceptores. En la interfaz `HttpInterceptor` que obliga a crear el método intercept que nos deja una firma un pelín compleja,  `intercept(req: HttpRequest<any>, next: HttpHandler):Observable<HttpEvent<any>>` . Vayamos por partes:

- `req: HttpRequest<any>` es un puntero a la peticíon en curso y su tipo deja bien claro que es eso, una http request.
- `next: HttpHandler` cada petición es manipulada por múltiples manejadores y cada interceptor es encadenado al siguiente manejador mediante este parámetro.
- `: Observable<HttpEvent<any>>`el tipo devuelto es llamativo porque además de ser un observable, resulta que usa un doble genérico. La razón es que este observable, no se queda simplemente con el evento de respuesta si no que observa cualquier evento http que le suceda a la petión. Entre ellos estará la recepción de la respuesta. Pero es uno más.

```typescript
  public intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    return next.handle(req)
  }
```
La respuesta del método se puede simplificar con `next.handle(req)`. Esto sería tener un interceptor que no hace abosolutamente nada. Habitualmente usaremos los interceptores para mutar la llamada, procesar la respuesta o ambas como en este caso.

```typescript
  public intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    const authReq = this.getReqWithAuthorization(req);
    return next.handle(authReq).pipe(tap(null, this.handleError));
  }
```

La llamada en curso no se puede modificar alegremente. Es inmutable, y por tanto, exige ser clonada si queremos hacer algún cambio. Afortunadamente Angular nos ofrece de métodos de clonación y mutado para las operaciones más habituales, como agregar una cabecera de autorización adjuntando un token a cada llamada. Claro que para que esto funcione `this.token` ha de tener un valor del que por ahora no tenemos ni idea.

```typescript
  private getReqWithAuthorization = (req: HttpRequest<any>) =>
    req.clone({
      setHeaders: { Authorization: `Bearer ${this.token}` }
    });
```

> En Angular promueven el uso de funciones y datos _inmutables_ de ahí que nos obliguen a clonar las cabeceras para modificarlas.

La otra gran labor encomendada a los interceptores es procesar las respuestas cuando lleguen desde el API. Para ello no hay más que entubar el operador RxJS adecuado en el stream de la petición. Por ejemplo un capturador genérico de errores con una especial atención al 401.

```typescript
  private handleError = err => {
    let userMessage = 'Fatal error';
    if (err instanceof HttpErrorResponse) {
      if (err.status === 401) {
        userMessage = 'Authorization needed';
      } else {
        userMessage = 'Comunications error';
      }
    }
    console.log(userMessage);
  };
```

> A parte de toda la _liturgia_ a la que nos obliga el `HttpInterceptor`, al final la lógica es sencilla. Se trata de rellenar la cabecera con el token actual. Si es o no válido es algo que decidirá el servidor. Aquí simplemente envías lo que tienes.

## 2.2 Enviar credenciales para obtener un token

Una correcta separación de responsabilidades no debe exigir nada más al interceptor. Debemos provverle de un mecanismo mediante el cual pueda notificar fallos de seguridad. Y de otro con el que pueda estar al tanto de cambios en el testigo de identificación del usuario. 

Esa labor la realizaremos en un servicio de intermediación. Hay muchas soluciones para este problema: un bus de enventos es una de ellas. Y un refinamiento del mismo es el patrón redux. Sin complicarnos demasiado, podemos empezar con una implementación *naive* como la del servicio `core/global-store.service.ts`

```typescript
export class GlobalStoreService {
  private state: GlobalState = { token: '', userMessage: '', loginNeeded: false };
  private token$ = new BehaviorSubject<string>(this.state.token);
  private userMessage$ = new BehaviorSubject<string>(this.state.userMessage);
  private loginNeeded$ = new BehaviorSubject<boolean>(this.state.loginNeeded);
  constructor() {}
  public selectToken$ = (): Observable<string> => this.token$.asObservable();
  public selectUserMessage$ = (): Observable<string> => this.userMessage$.asObservable();
  public selectLoginNeeded$ = (): Observable<boolean> => this.loginNeeded$.asObservable();
  public dispatchToken = (token: string) => {
    this.state.token = token;
    this.token$.next(this.state.token);
  };
  public dispatchUserMessage = (userMessage: string) => {
    this.state.userMessage = userMessage;
    this.userMessage$.next(this.state.userMessage);
  };
  public dispatchLoginNeeded = (loginNeeded: boolean) => {
    this.state.loginNeeded = loginNeeded;
    this.loginNeeded$.next(this.state.loginNeeded);
  };
}
```

Sin entrar en mucho detalle, por ahora, esta clase es un intermediario que permite comunicar objetos despachando operaciones y recibir notificaciones subscribiendonos a temas de interés seleccionados.

En el caso de la seguridad, el interceptor notifica los fallos y recibe los cambios en el token. Este es un extracto de la funcionalidad usada en el `AuthInterceptorService`.

```typescript
  private token: string;
  constructor(private globalStore: GlobalStoreService) {
    this.globalStore.selectToken$().subscribe((token: string) => (this.token = token));
  }
  private handleError = err => {
    if (err instanceof HttpErrorResponse && err.status === 401) {
       this.globalStore.dispatchLoginNeeded(true);
    }
  };
```

En algún lugar de la ráiz de la plcación, como el `core/vavigator.componente.ts` es necesario escuchar ciertos eventos  y redirigir al usuario convenientemente.

```typescript
export class NavigatorComponent implements OnInit {
  constructor(private globalStore: GlobalStoreService, private router: Router) {}
  ngOnInit() {
    this.globalStore.selectLoginNeeded$().subscribe(loginNeeded => {
      if (loginNeeded) {
        this.router.navigateByUrl('/auth');
      } else {
        this.router.navigateByUrl('/');
      }
    });
  }
}
```

Por supuesto que necesitaremos un formulario que recoja las credenciales del usuario y la envíe al servidor para validar y su cusao recibir un token. Pero todo eso lo tienes en el repositorio de ejemplo. Aquí simplemente pongo la pieza que cierra el círulo de la seguridad, la recepcióndel token en `auth/access.component.ts`

```typescript
export class AccessComponent implements OnInit {
  constructor(private http: HttpClient, private globalStore: GlobalStoreService) {}
  public onLogin() {
    this.http.post(this.url, this.credentials)
      .subscribe(authResponse => this.globalStore.dispatchToken(authResponse.token));
  }
}
```

Si se aceptan las credenciales, **el servidor nos devolverá un objeto con el _token_ de la sesión** para el usuario. Es habitual que envíe más información como roles, y preferencias del usuario... pero eso ya depende del API. Lo que depende de ti es guardar ese _token_.

El **almacenamiento recomendado en los navegadores es el `localStorage` o el `sessionStorage`** pero en este tutorial introductorio tendrás que conformarte con almacenarlo en la memoria. 

# 3 Guardias

`car/travel.guard.ts`
*Work in progress...*

---

Ya tenemos al usuario identificado y los datos se envían o reciben acompañados de una cabecera que el servidor interpreta como una firma; lo básico para un sistema mínimamente seguro. Sigue esta serie para crear tus [formularios reactivos con Angular](../formularios-reactivos-con-Angular/) mientras aprendes a programar con Angular6. 


> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>
