---
title: Páginas y rutas Angular SPA
permalink: paginas-y-rutas-angular-spa
date: 2019-01-21 13:41:14
tags:
  - Angular
  - SPA
  - Routing
  - Tutorial
  - Introducción
  - Angular7
  - Angular2
categories:
  - [Tutorial, Angular]
thumbnail: /css/images/angular-2_spa.png
---

![paginas-y-rutas-angular-spa](/images/tutorial-angular-2_spa.png)

Las **aplicaciones Angular 7 son conjuntos de páginas enrutadas** en el propio navegador. Son las conocidas _SPA, Single Page Applications_. Estas apps liberan al servidor de una parte del trabajo, reducen la cantidad de llamadas y mejoran la percepción de velocidad del usuario.

En este tutorial aprenderás a crear una Angular SPA fácilmente usando `@angular/router`, **el enrutador de Angular**.

<!-- more -->

Partiendo de la aplicación tal cómo quedó en [Base para una aplicación Angular](../base-aplicacion-angular/). Seguimos usando el concepto de árbol, ahora como analogía de **las rutas y las vistas** asociadas. Al finalizar tendrás una angular SPA con vistas asociadas a sus rutas.

> Código asociado a este artículo en _GitHub_: [AcademiaBinaria/angular-board/](https://github.com/AcademiaBinaria/angular-board/)

# 1. Rutas

Al crear la aplicación hice uso del flag `routing true` en el comando de generación del _CLI_. Esto causó la aparición de no uno, sino dos módulos gemelos en la raíz de la aplicación. Has estudiado el `AppModule` verdadero módulo raíz, y ahora verás en profundidad a su gemelo: el **módulo de enrutado** `AppRoutingModule` y el uso que hace del `RouterModule`.

## 1.1 RouterModule

El _Angular Router_ necesita ser importado y configurado. El módulo `AppRoutingModule` cumple dos funciones. Por un lado **importa al `RouterModule`** de Angular, el cual contiene toda la lógica necesaria para enrutar en el navegador. Por otro lado, permite la **definición de rutas** en el array `Routes[]`.

```typescript
import { Routes, RouterModule } from '@angular/router';
const routes: Routes = [
  {
    path: 'not-found',
    component: NotFoundComponent
  },
  {
    path: '**',
    redirectTo: 'not-found'
  }
];
@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule {}
```

El array de rutas recibe **objetos ruta** con propiedades de configuración.

La primera es `path:` en la que se especifica **la dirección** que resuelve, en este caso la ruta vacía o raíz del árbol de rutas. Las otras son opcionales y las veremos poco a poco.

### 1.1.1 Component

Empecemos casi por el final y de paso hagamos algo útil para no perdernos. Un detector de rutas no contempladas, y una ruta a dónde redirigir a los usuarios perdidos. Para ello estudiaremos la propiedad `component` que es fundamental pues indica **el componente** que se debe mostrar cuando esta ruta se active.

Así es cómo funciona el enrutado. Un camino y un componente asociado. La tabal de enrutado se procesa de arriba a abajo y cuando un camino coincide con la ruta actual, se para y se carga el componente.

Vamos a crear un componente con la intención de mostrarlo sólo cuando las demás ruta fallen. Se llamará _not found_

```bash
ng g c core/not-found
```

Ya podemos asociar dicho componente al camino `not-found`. Pero esto es poca cosa. Hay mucho más.

### 1.1.2 RedirectTo

La configuración de rutas no sólo permite asignar componentes a las direcciones. También se pueden hacer **redirecciones de unas direcciones a otras**. Y por supuesto puede haber **rutas no contempladas o errores** por parte del usuario, los infames `404 Not Found`.

> En este caso cuando se escriba la ruta `/not-foud` se mostrará un componente, el `NotFoundComponent`, cuyo contenido indicará al usuario que se ha perdido. Claro que nadie va voluntariamente a esa ruta. Mediante el `path: '**'` le indico que ante cualquier ruta no contemplada anteriormente se ejecute el comando `redirectTo: 'not-found'`, el cual nos lleva a una ruta conocida con un mensaje bien conocido. _Page Not Found_.

Pero ¿Cómo es eso de que se mostrará?, ¿Dónde se cargará?. Presentamos a `<router-outlet>`.

## 1.2 Router Outlet

La idea general de **una SPA es tener una única página que cargue dinámicamente otras vistas**. Normalmente la página contenedora mantiene el menú de navegación, el pie de página y otras áreas comunes. Y deja un espacio para la carga dinámica. Para ello necesitamos saber **qué componente cargar y dónde mostrarlo**. De esto último se ocupa el _router outlet_ mediante la etiqueta `<router-outlet></router-outlet>`.

En el `main.component.ts` había un contenido _hard-coded_. Vamos a mantenerlo hasta que encontremos un sitio. Pero comentado, para que no se vea. Para hacer que el contenido sea dinámico se sustituye por el elemento de Angular `<router-outlet></router-outlet>`. Este elemento del framework inyectará dinámicamente el componente que le corresponda según la ruta activa. El `MainComponent` queda así:

```typescript
    selector: 'app-main',
    template: `
      <main class="container ">
        <router-outlet></router-outlet>
      </main>
    `,
    styles: []
```

## 1.3 Router Link

Los enlaces web tradicionalmente se han resuelto con elementos `<a href=""></a>` dónde en su atributo `href` se asociaba la dirección a la cuál navegar ante el click del usuario. **En Angular los enlaces se declaran con un atributo** especial llamado `routerLink`. Este atributo **se compila dando lugar al `href`** oportuno en tiempo de ejecución.

En el fichero `not-found.component.ts` pon algo así:

```typescript
  selector: "cf-not-found",
  template: `
    <h1>Not Found</h1>
    <h2>404</h2>
    <a routerLink="/">Go home</a>
  `,
  styles: []
```

> Por ahora la funcionalidad de `routerLink` no mejora en nada a `href`. Pero lo hará. Mientras tanto familiarízate con su sintaxis y... asegúrate de importar `RouterModule` en los módulos en los que lo vayas a usar.

Salgamos de este bucle creando más rutas y más componentes. Pero esta vez con un nueva técnica.

# 2 Lazy Loading

La web clásica funcionaba con un navegador pidiendo una ruta al servidor. El servidor buscaba o montaba un documento html y se l devolvía al navegador para que lo renderizase. Una nueva ruta significaba repetir todo ese viaje. Hasta que aparecieron las [Single Page Applications](https://en.wikipedia.org/wiki/Single-page_application). En este caso el código cliente es el responsable del contenido asociado a cada ruta. Y eso es mucha responsabilidad.

Las _webs SPA_ se crearon por una razón que casi acaba con ellas: **la velocidad**. Al realizar el enrutado en el cliente y querer evitar todos los viajes posibles hasta el servidor, se cargó a la única página web con todo el peso de la aplicación. Lo cual la hizo terriblemente lenta en la primera visita de cada usuario.

El **impacto de la primera visita** en una aplicación de intranet no suele ser un problema grave. Pero en internet esa visita puede ser la primera y si tarda mucho, también será la última. La solución viene de mano del concepto de _lazy loading_ o carga perezosa. Consiste en diferir la carga de la lógica asociada a una dirección hasta el momento en que sea activada dicha ruta. De esa forma, **una página no visitada es una página que no pesa**. Y la carga inicial se hace mucho más liviana.

En Angular el _lazy loading_ es tan sencillo que ya se recomienda implementarlo por defecto. Para hacerlo conoceremos más comandos del `Router` y algunas herramientas de compilación usadas por el _Angular CLI_.

## 2.1 Webpack y los bundles por ruta

Hay que saber que el _Angular CLI_ usa internamente la herramienta de empaquetado _webpack_. La cual recorre el código _TypeScript_ buscando `imports` y empaquetando su contenido en sacos o _bundles_. Luego introduce las referencias a esos _bundles_ en la sección se _scripts_ del index.html, haciendo que se descarguen todos nada más arrancar la aplicación. Esto puede ser muy pesado en aplicaciones grandes. Así que hay que buscar una manera de diferir esa descarga, repartiendo el _bundle_ principal en otros más pequeños que se cargará bajo demanda.

Objetivo: adelgazar el peso del _bundle_ principal, el `main.js`.Para conseguirlo hay que configurar las rutas de forma que no sea necesario importar los componentes a mostrar. Tal como se ha hecho con el `NotFoundComponent`, de hacerlo así con todos, _webpack_ empaquetaría esos componentes como algo necesario... y por tanto serían enviados al navegador en el _bundle_ principal sin que sea seguro su uso. Ese no es el camino, es una excepción para componente poco pesado y muy utilizados.

La solución que ofrecen el _cli_ y _webpack_ consiste en **delegar la asignación del componente a otro módulo, pero sin importarlo** hasta que su ruta principal se active.

He creado unas vistas para ser usadas en las direcciones `/` y `/about`. Los componentes asociados se llaman `HomeComponent` u `AboutComponent`. Se han **declarado pero no exportado** en sus respectivos módulos `HomeModule` y `AboutModule`. No es necesario exportarlos porque no serán reclamados directamente por nuestro código.

```bash
ng g m home --routing true
ng g c home/home
ng g m about --routing true
ng g c about/about
```

Estos módulos no deben ser importados por el `AppModule`; no queremos saber de su existencia. Simplemente debe usarse su ruta relativa en el módulo de enrutado `AppRoutingModule` como un valor especial. Vamos a agregarlo al `app-routing.module.ts` que quedará así.

```typescript
import { Routes, RouterModule } from '@angular/router';
const routes: Routes = [
  {
    path: '',
    loadChildren: './home/home.module#HomeModule'
  },
  {
    path: 'about',
    loadChildren: './about/about.module#AboutModule'
  },
  {
    path: 'not-found',
    component: NotFoundComponent
  },
  {
    path: '**',
    redirectTo: 'not-found'
  }
];
@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule {}
```

Fíjate que **la dirección del fichero es una cadena de texto** asignada a una nueva propiedad de objeto _route_, la propiedad `loadChildren:""`. No se está produciendo ninguna importación en _TypeScript_ como ocurre con el componente `NotFoundComponent`.

Con esta información _webpack_ va a generar un _bundle_ específico para cada módulo. Si durante la ejecución se activa la ruta `/` (muy probable porque es la ruta raíz) o la ruta `/about` entonces se descarga ese paquete concreto y se ejecuta su contenido. Mientras tanto, se queda almacenado en el servidor.

> Esto hace que la aplicación de Angular pese menos y responda antes, mejorando el tiempo de pintado inicial. La combinación de estas y otras técnicas que veremos en este tutorial sacarán el mejor rendimiento posible a tu aplicación Angular.

## 2.2 El enrutador delegado

Ya sabemos que hasta que no se active la ruta `/` o la `/about` no hay que hacer nada. Pero si se activa, entonces se descarga un _bundle_ que contiene un módulo y los componentes necesarios. Sólo falta escoger dentro de ese módulo el componente que se asignará a la ruta.

Para eso al crear el módulo de operaciones usé el _flag_ `routing true`. Esto hace que se genere un segundo módulo de enrutado. El `HomeRoutingModule` y el `AboutRoutingModule` son prácticamente idénticos al enrutador raíz.

> Digamos que son **enrutadores subordinados** al primero. Sólo se llega aquí si en la ruta principal se ha navegado a una dirección concreta. Se hace notar esa distinción durante el proceso de importación del módulo de Angular `RouterModule`. En el caso principal se pone `imports: [RouterModule.forRoot(routes)]` y en todos los demás `imports: [RouterModule.forChild(routes)]`.

A nivel subordinado, la dirección `path: ""` se agrega al `path: ""` de su enrutador padre. Cuidado, es un error común repetir el _path_ a nivel hijo. En este caso incluso parece redundante. Pero con _about_ no quedan dudas. En el _root_ lleva `path: "about"` y en el _child_ solamente `path: ""`.

La ventaja real de este segundo enrutador es que irá empaquetado en el mismo _bundle_ que el módulo de negocio y sus componentes. Descargando ese peso en el momento que se necesite. Aquí sí que asignaremos un componente concreto: el `HomeComponent` o el `AboutComponent`. Por ejemplo el fichero `home-routing.module.ts` quedará más o menos así:

```typescript
import { HomeComponent } from './home/home.component';
const routes: Routes = [
  {
    path: '',
    component: HomeComponent
  }
];
@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class HomeRoutingModule {}
```

## 2.3 Navegación

Ahora que ya tenemos un par de rutas reales, es buen momento para crear un mini menú de navegación. Vayamos al `core/shell/header.component.html` y pongamos algo así:

```html
<header class="sticky">
  <a routerLink="/" class="logo"> <span class="icon-home"></span> <span>{{ title }}</span> </a>
  <a routerLink="about" routerLinkActive="router-link-active" class="button">
    <img width="32" style="vertical-align: -0.5em" src="./assets/logo.png" /> <span> About us</span>
  </a>
</header>
```

# 3 Parámetros

Las rutas vistas hasta ahora se consideran estáticas pues se han definido usando constantes. Es muy habitual tener **páginas con la misma estructura pero distintos contenidos**. Un blog con sus posts, una tienda con sus productos, o un proyecto con sus tareas... hay miles de ejemplos así.

Ese tipo de direcciones se consideran paramétricas, tienen unos segmentos estáticos y otros dinámicos. Estos últimos se definen con parámetros, algo así como **variables dentro de la cadena de la ruta**. Su sintaxis obliga a precederlas de dos puntos. Por ejemplo `countries/:country/cities/:city` resolvería rutas como _countries/usa/cities/new-york_ o _countries/italy/cities/roma_. Rellenando los parámetros `:country` y `:city` con los valores necesarios.

> Esta aplicación no tiene un propósito de negocio concreto. Iremos creando rutas según sea necesario por motivos pedagógicos. Empezaremos con unas páginas destinadas a gestionar proyectos.

Vamos a crear rutas como _/projects/write-a-book_ o _/projects/develop-a-personal-web_. Para ello necesitamos el segmento principal _/projects_ y delegar su uso al `ProjectsModule`.

Creamos el módulo y sus componentes principales como hasta ahora.

```bash
ng g m projects --routing true
ng g c projects/projects
ng g c projects/projects/new-project
ng g c projects/projects/project
```

En las rutas del `app-routing.module.ts` agregamos un nuevo _path lazy_

```typescript
const routes: Routes = [
...
  {
    path: 'projects',
    loadChildren: './projects/projects.module#ProjectsModule'
  },
...
];
```

Y en el recién creado `projects\projects-routing.module.ts` asignamos las rutas a los componentes.

```typescript
const routes: Routes = [
  {
    path: '',
    component: ProjectsComponent
  },
  {
    path: 'new',
    component: NewProjectComponent
  },
  {
    path: ':id',
    component: ProjectComponent
  }
];
```

Esta configuración resuelve las rutas `projects/`, `projects/new` y `projects/cualquier-otra-cosa`. Y las carga con el componente adecuado. Lo novedoso en el camino `:id`. El prefijo dos puntos indica que es un parámetro. Algo así como una variable en el segundo segmento que se almacenará y será recogido con el arbitrario nombre `id`.

Para mostrar el uso de los nuevos enlaces he agregado el `projects/` al `HeaderComponente` y he creado un listado en el `ProjectsComponent`. Familiarízate con las rutas relativas para componer la ruta completa. La parte interesante de su _html_ es:

```html
<header class="sticky">
  <a routerLink="./" class="button"> <span> Projects</span> </a>
  <a routerLink="new" class="button"> <span> New</span> </a>
</header>
<h2>List</h2>
<ul>
  <li>
    <h4><a routerLink="write-a-book">Write a Book</a></h4>
  </li>
  <li>
    <h4><a [routerLink]="['develop-a-personal-web']">Develop a personal web</a></h4>
  </li>
</ul>
```

Aún más interesante es el componente que muestra cada proyecto de la lista, el `ProjectComponent`. En este caso fíjate cómo accede a la ruta, cómo obtiene el valor del parámetro y cómo lo usa para mostrarlo en la web.

Contenido del fichero `project.component.ts` relacionado con la obtención del parámetro de la ruta activa:

```typescript
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

export class ProjectComponent implements OnInit {
  public projectId = '';
  constructor(activateRoute: ActivatedRoute) {
    this.projectId = activateRoute.snapshot.params['id'];
  }
  ngOnInit() {}
}
```

## 3.1 ActivatedRoute

El framework _Angular_ trae muchas librerías para facilitar la vida al programador. Sólo hay que saber dónde están y cómo pedirlas. Para ello volvemos a la tecnología escogida, _TypeScript_, que permite las **importaciones y la inyección de dependencias**. Hay un tema dedicado a conocer en profundidad [los servicios inyectables en Angular](https://academia-binaria.com/servicios-inyectables-en-Angular/). Por ahora una breve introducción.

La instrucción `import { ActivatedRoute } from "@angular/router";` pone a disposición del programador el código donde está definida la clase `ActivatedRoute`. Pero no se instancia directamente; en su lugar, se usa como un argumento del constructor de la clase del componente. Ese constructor es invocado por _Angular_, y dinámicamente el propio framework sabe cómo rellenar los argumentos que se pidan en los constructores. Es decir, sabe cómo inyectar instancias en las que dependencias declaradas.

Una vez que han **inyectan las dependencias en el constructor** ya están listas para ser usadas. En concreto `activateRoute` da acceso a métodos y propiedades para trabajar con la ruta activa y poder leer sus parámetros.

Obtenidos los datos desde la _URL_, ya se muestran en la vista de forma ya conocida. Fichero `projects/project.component.html`

```html
<h2>The project</h2>
<h3>{{ projectId }}</h3>
```

# 4 Rutas anidadas

Cuando las interfaces se complican, es habitual que las aplicaciones dispongan de menús de navegación a distintos niveles. Dentro de una misma página podemos querer ver distinto contenido y además reflejarlo en la _URL_. Para resolver esta situación en Angular disponemos de la técnica de las _nested routes_.

> De una manera un tanto forzada la he incluido en la página `/about`. La cual disponen de su propio menú de navegación, y lo que es más importante, su propio `<router-outlet></router-outlet>`.

Para empezar veamos como queda el _html_ del `projects.component.ts`. Vamos a dotarlo de dos rutas nuevas `/about/links` y `/about/info`. Cada una mostrará contenido en un componente adecuadamente insertado en el `<router-outlet></router-outlet>` local.

Para repasar conceptos de generación de componentes

```bash
ng g c about/about/links
ng g c about/about/info
```

Para que funcione empezamos por crear los dos componentes `LinksComponent` e `InfoComponent` de forma rutinaria. Y los asignamos en el `about-routing.module.ts` como subordinados a la ruta principal con el comando `children:[]`. Los caminos se van agregando sobre la ruta principal activa, la `/about`. Esto es así tanto el `routerLink` como el `path` de los `children`.

```typescript
const routes: Routes = [
  {
    path: '',
    component: AboutComponent,
    children: [
      {
        path: 'links',
        component: LinksComponent
      },
      {
        path: 'info',
        component: InfoComponent
      }
    ]
  }
];
```

Los componentes de las rutas `children` se inyectarán en el `<router-outlet>` del componente contenedor `AboutComponent`. Es como si todo volviese a empezar desde aquí.

```html
<header class="sticky">
  <a routerLink="links" class="button"> <span> Tutorial Links</span> </a>
  <a routerLink="info" class="button"> <span> More Info</span> </a>
</header>
<router-outlet></router-outlet>
```

Con estos conceptos y la combinación de `children, loadChild, component, redirectTo` ... asociadas a `path` podrás configurar tu aplicación y responder a cualquier _URL_ desde la misma y única página `index.html`.

Con esto tendrás una aplicación SPA en _Angular_. Sigue esta serie para añadirle [Formularios, tablas y modelos de datos en Angular](../formularios-tablas-y-modelos-de-datos-en-angular/) mientras aprendes a programar con Angular6.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>
