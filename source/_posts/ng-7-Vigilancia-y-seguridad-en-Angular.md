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


# 1 Seguridad

La seguridad de las comunicaciones con un servicio REST se resuelve habitualmente mediante una **credencial generada por el servidor llamada _token_**. Un usuario registrado en el sistema puede hacer _log in_ enviando una vez su identificador y contraseña. Si todo va bien, a cambio el servidor le enviará un _token_ que deberá usar en las siguientes llamadas. Con esto el servidor será capaz de autentificar las llamadas y responder adecuadamente.

## 1.1 Detectar intrusos

En el ejercicio anterior usé el `CatchInterceptorService` para capturar los errores obtenidos del servidor. Cuando me llegue un código `401 Unauthorized` querrá decir que el servidor no acepta las actuales credenciales del usuario. Lo que hago es llevar al usuario a una página para que pueda registrarse o volver a identificarse en el sistema.

```typescript
private catchHttpError(err: HttpErrorResponse) {
  if (err.status === 401) {
    this.catchUnauthorized();
  } else {
    console.warn(err.statusText);
  }
}
private catchUnauthorized() {
  console.log("Not authorized");
  this.navigateToLogin();
}
private navigateToLogin() {
  this.router.navigateByUrl("/credentials/login");
}
```

## 1.2 Obtener credenciales

Mediante un formulario pregunto al usuario los datos de identificación estándar: _email y password_. Estos se envían al servidor para que registre un usuario nuevo o valide a uno existente según el caso. Mira el código del fichero `credentials.component.ts`.

```typescript
public sendCredential() {
  this.errorMessage = "";
  const credential = this.pageData.credential;
  const service = this.pageData.title;
  this.credentialsService
    .sendCredential(credential, service)
    .subscribe(
      this.acceptedCredentials.bind(this),
      this.invalidCredentials.bind(this)
    );
}
private acceptedCredentials(token) {
  this.busService.emitUserToken(token);
  this.router.navigateByUrl("/");
}
private invalidCredentials() {
  this.busService.emitUserToken(null);
  this.errorMessage = "Invalid Credentials";
}
```

> Este componente sirve para registrar o identificar usuarios. Cambia su comportamiento según el valor de `this.pageData` que viene determinado desde el enrutador. Esta es una manera sencilla de reutilizar componentes. Mira en `credentials.routing.ts` para tener más detalles.

## 1.3 Almacenamiento del token

Si se aceptan las credenciales **el servidor nos devolverá un objeto con el _token_ de la sesión** para el usuario. Es habitual que envíe más información como roles, y preferencias del usuario... pero eso ya depende del API. Lo que depende de ti es guardar ese _token_.

El **almacenamiento recomendado en los navegadores es el `localStorage`** pero en este tutorial introductorio tendrás que conformarte con almacenarlo en la memoria. Eso sí, necesitamos un lugar que sea accesible para un interceptor que aún no has visto: el `TokenInterceptorService`, que se encargará de enviar dicho _token_ en todas las llamadas. Para comunicar este componente de las credenciales con ese interceptor sin acoplarlos he decidido usar un servicio intermedio: el `BusService`.

### 1.3.1 El bus service

Este servicio del fichero `bus.service.ts` es la implementación más sencilla del patrón _Redux_ que he podido crear. Se basa en utilizar la librería `RxJs` para emitir cambios en el estado de un modelo; y que otro servicio pueda subscribirse para ser notificado de dichos cambios.

El emisor será el componente `CredentialsComponent` que envía las credenciales al servidor y recibe el _token_. El subscriptor será el servicio de interceptación  `TokenInterceptorService` que usará dicho _token_ para identificar al usuario actual en todas las llamadas al servidor. Y en el medio está el `BusService` que actúa de enlace entre ambos. Este es el código necesario en el fichero `bus.service.ts`:

```typescript
private userToken$ = new Subject<any>();

constructor() {}

public getUserToken$(): Observable<any> {
  return this.userToken$.asObservable();
}
public emitUserToken(userToken: any) {
  this.userToken$.next(userToken);
}
```

El tipo genérico `Subject<any>` viene en la librería `rxjs/Subject` y es el hermano mayor del ya conocido `Observable<any>`. En este caso permite ademas _emitir_ valores que recibirán los subscriptores. La suscripción puede realizarse directamente contra la instancia del `Subject`, pero lo recomendable es que dicha instancia sea privada y que sólo exponga una parte de su funcionalidad. 
> Digamos que exponemos el _Observable_ de sólo lectura obtenido mediante la función `.asObservable()`.

### 1.3.2 El Token Interceptor Service

Ya sólo falta consumir ese _Observable_ en el servicio interceptor `token-interceptor.service.ts`. Para ello me suscribo a los cambios emitidos desde el `BusService` y guardo el token que me envíen para su uso posterior.

```typescript
private token: string = "InitialAuthorizationToken";

constructor(private busService: BusService) {
  this.subscribeToTokenChanges();
}

private subscribeToTokenChanges() {
  this.busService.getUserToken$().subscribe(this.setTokenIfAny.bind(this));
}
private setTokenIfAny(data) {
  if (data && data.token) {
    this.token = data.token;
  }
}
```

# 2 vigilancia

El servicio `TokenInterceptorService` se encarga de enviar el _token_ actual en cada llamada que pasa por sus manos. Para ello implementa la interfaz `HttpInterceptor` en su método `intercept()` con la lógica suficiente para enviar el token en una cabecera acordada con el API. En este caso uso la estándar `Authorization`.

```typescript
public intercept(
  req: HttpRequest<any>,
  next: HttpHandler
): Observable<HttpEvent<any>> {
  const authorizationReq = this.setAuthHeader(req);
  const handledRequest = next.handle(authorizationReq);
  return handledRequest;
}
private setAuthHeader(req: HttpRequest<any>): HttpRequest<any> {
  const authorization = `Bearer ${this.token}`;
  const headers = req.headers.set("Authorization", authorization);
  const authorizationReq = req.clone({ headers });
  return authorizationReq;
}
```

A parte de toda la _liturgia_ a la que nos obliga el `HttpInterceptor`, al final la lógica es sencilla. Se trata de rellenar la cabecera con el token actual. Si es o no válido es algo que decidirá el servidor. Aquí simplemente envías lo que tienes.

> En Angular promueven el uso de funciones y datos _inmutables_ de ahí que nos obliguen a clonar las cabeceras para modificarlas.

Ya tenemos al usuario identificado y los datos se envían o reciben acompañados de una cabecera que el servidor interpreta como una firma; lo básico para un sistema mínimamente seguro. Sigue esta serie para crear tus [formularios reactivos con Angular](../formularios-reactivos-con-Angular/) mientras aprendes a programar con Angular6. 


> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>
