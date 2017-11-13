---
title: Páginas y rutas Angular SPA
permalink: paginas-y-rutas-angular-spa
date: 2017-11-13 17:19:14
tags:  
- Angular
- Angular5
- SPA
- Tutorial
categories:
- [Tutorial, Angular] 
thumbnail: /css/images/angular-5_2_SPA.jpg
---
![Tutorial Angular5 1-Base](/images/tutorial-angular-5_2_SPA.jpg)

Las **aplicaciones Angular son conjuntos de páginas enrutadas** en el propio navegador. Son las conocidas *SPA, Single Page Applications*. Estas apps liberan al servidor de una parte del trabajo, reducen la cantidad de llamadas y mejoran la percepción de velocidad del usuario.

Seguimos usando el concepto de árbol, ahora como analogía de **las rutas y las vistas** asociadas. Algo que se consigue fácilmente con `@angular/router` **el enrutador de Angular**.

<!-- more -->

Partiendo de la aplicación tal cómo quedó en [Base para una aplicación Angular](../base-aplicacion-angular/). Al finalizar tendrás un SPA con vistas asociadas a sus rutas.

>Código asociado a este artículo en *GitHub*: [AcademiaBinaria/angular5/2-spa](https://github.com/AcademiaBinaria/angular5/tree/master/2-spa/cash-flow) 


# 1. Rutas
Al crear la aplicación hice uso del flag `routing true` en el comando de generación del *CLI*. Esto causó la aparición de no uno, sino dos módulos gemelos en la raíz de la aplicación. Has estudiado el `AppModule` verdadero módulo raíz, y ahora verás el 'AppRoutingModule'.

## 1.1 RouterModule
Este módulo cumple dos funciones. Por un lado importa al `RouterModule` que contiene toda la lógica necesaria para enrutar en el navegador. Por otro lado, permite la definición de rutas en el array `Routes[]`. 

>Por motivos estéticos he cambiado el nombre original del fichero `app-routing.module.ts` a `app.routing.ts` y así disponer de un icono propio en el tema [Material Icon Theme](https://marketplace.visualstudio.com/items?itemName=PKief.material-icon-theme).

```typescript
import { Routes, RouterModule } from "@angular/router";
import { HomeComponent } from "./views/home/home.component";
const routes: Routes = [
  {
    path: "",
    component: HomeComponent
  }];
@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule {}
```

El array de rutas recibe objetos con propiedades de configuración. La principal es `path:` en la que se especifica la dirección que resuelve, es este caso la ruta vacía o raíz del árbol de rutas. Otra propiedad fundamental es `component` la cual indica el componente que se debe mostrar cuando esta ruta se active.

En este caso he aprovechado el componente `HomeComponent` para asociarlo a la ruta raíz. Pero, ¿qué pasará con dicho componente? ¿dónde se cargará?. Presentamos a `<router-outlet>`.

## 1.2 Router Outlet 
La idea general de una SPA es tener una única página que cargue dinámicamente otras vistas. Normalmente la página contenedora mantiene el menú de navegación, el pie de página y otras áreas comunes. Y deja un espacio para la carga dinámica. Para ello necesitamos sabér qué comonente cargar () y dónde mostrarlo. De esto último se ocupa la etiqueta ` <router-outlet></router-outlet>`.

En el `app.component.ts` había un reclamo directo al componente `cf-home`. Para hacerlo dinámico se sustituye por ` <router-outlet></router-outlet>` quedando algo así:

```typescript
    selector: "cf-root",
    template: `
      <cf-nav></cf-nav>
      <router-outlet></router-outlet>
      <cf-footer></cf-footer>
    `
    styles: []
```
> Puedes ver los cambios realizados en [este *commit*](https://github.com/AcademiaBinaria/angular5/commit/a0ae9077ea2a74e8683de8b281147661b7a9f508)

## 1.3 Router Redirect
La configuración de rutas no sólo permite asignar compoentes a las direcciones. También se pueden hacer redirecciones de unas a otras direcciones. Y por supuesto, puede haber rutas no contempladas o errores por parte del usuario, los infames `404 Not Found`.

Un ejemplo de ambos sería configurar nuestras rutas de forma que toda ruta desconocida nos lleve a otra general que muestre un mensaje predeterminado. Para eso crea un módulo y un componente llamados `NotFound` con los siguientes comandos:

```shell
ng g m views/not-found
ng g c views/not-found/not-found --export --flat
```
Vuelve al módulo de enrutado ahora conocido como `app.routing.ts` agrega dos nuevas entradas al array `routes`.

```typescript
const routes: Routes = [
  {
    path: "",
    component: HomeComponent
  },
  {
    path: "404",
    component: NotFoundComponent
  },
  {
    path: "**",
    redirectTo: "/404"
  }
  ];
```

La entrada interesante es la última. Su dirección `path: "**"` indica que es cualquier ruta que no haya sido resuelta previamente. Un *not found* de toda la vida. En este caso lo redirijo a una ruta existente: `/404`. Y a esta última se le asocia un componente concreto, el `NotFoundComponent`.

## 1.4 Router Link
Los enlaces wen tradicionalmente se han resuelto con elementos `<a></a>` dónde en su atributo `href` se asociaba la dirección a la cuál navegar ante el click del usuario. En Angular los enlaces se declaran con un atributo especial llamada `routerLink`. Este atributo se compila dando lugar al `href` oportuno.

En el fichero `not-found.component.ts` pon algo así:
```typescript
  elector: "cf-not-found",
  template: `
    <h1>not Found</h1>
    <h2>404</h2>
    <a routerLink="/">Go home</a>
  `,
  styles: []
```

>Por ahora la funcionalidad de `routerLink` no mejora en nada a `href`. Pero lo hará. Mientras tanto familiarízate con su sintaxis y... asegúrate de importar `RouterModule` en `not-found.module.ts`. Puedes ver los cambios realizados en [este *commit*](https://github.com/AcademiaBinaria/angular5/commit/464a5f0fd5f8975157793b0a3c13d7f61f890fa5) 


# 2 Lazy Routing
Las webs SPA se crearon por una razón que casi acaba con ellas: la velocidad. Al realizar el enrutado en el cliente, y querer evitar todos los viajes posibles hasta el servidor, se cargó a la única página web con toda la carga de la aplicación. Lo cual la hizo terriblemente lenta en la primera visita de una usuario. 

El impacto de la primera visita en una aplicación de intranet no suele ser un problema. Pero en internet esa visita puede ser la primera y la última. La solución viene de mano del concepto de *lazy loading* o carga perezosa. Consiste en diferir la carga de la lógica asociada a una dirección hasta el momento en que sea activada dicha ruta. De esa forma, una página no visitada es una página que no pesa. Y la carga inicial se hace mucho más liviana.

## 2.1 Webpack y los bundles por ruta
Para conseguirlo hay que declarar la ruta de forma que no sea necesario importar el componete a mostrar. De otro modo *webpack* empaquetaría ese componente como algo necesario... y por tanto sería enviado al navegador en el *bundle* principal sin que sea seguro su uso. La solución que ofrecen el *cli* y *webpack* consiste en delegar la asignación del componente a otro módulo, pero sin importarlo.

He creado un una nueva vista en una nueva dirección llamada `/operations`. El componente `OperationsComponet` dónde se ha declarado pero no exportado en el módulo `OperationsModule`. 

```shell
ng g m views/operations --routing true
ng g c views/operations/operations --flat
```

Este módulo no debe ser importado por el `AppModule`. Simplemente debe usarse su ruta relativa en el módulo de enrutado `AppRoutingModule` como un valor especial:

```typescript
const routes: Routes = [
  {
    path: "",
    component: HomeComponent
  },
  {
    path: "operations",
    loadChildren: "./views/operations/operations.module#OperationsModule"
  },
  {
    path: "404",
    component: NotFoundComponent
  },
  {
    path: "**",
    redirectTo: "/404"
  }
  ];
```
Con esta información *webpack* va a generar un *bundle* específico para este módulo. Si durante la ejecución se activa la ruta `/operations` entonces descarga ese paquete y ejecuta su contenido.

## 2.2 El enrutador delegado
Ya sabemos que hasta que no se active la ruta `/operations` no hay que hacer nada. Pero si se activa, entonces se descarga un *bundle* que contiene un módulo y los componentes necesarios. Sólo falta escoger el componente que se asignará a la ruta.

Para eso al crear el módulo operaciones usé el *flag* `routing true`. Esto hace que se genere un segundo módulo de enrutado. El `OperationsRoutingModule` prácticamente idéntico al enrutador raíz. Digamos que es un enrutador subordinado al primero,al cual sólo se llega si en la ruta principal se ha navegado a una dirección concreta. A este nivel la dirección `path: ""` equivale al `path: "operations"` de su enrutador padre.

La ventaja real de este segundo enrutador es que irá empaquetado en el mismo *bundle* que el módulo de negocio y sus componentes. Así que aquí sí que asignaremos un componente concreto. El `OperationsComponent` dejando el fichero `operations.routing.ts` más o menos a´si:

```typescript
import { OperationsComponent } from "./operations.component";
const routes: Routes = [
  {
    path: "",
    component: OperationsComponent
  }
];
```

>Puedes tener una idea general de los cambios realizados en [este *commit*](https://github.com/AcademiaBinaria/angular5/commit/b3f79c3324561174e10a7f8fb2668d7dc3e610f1) 


# 3 Parámetros
Rutas con parámetros... 
... Work in progress ...
