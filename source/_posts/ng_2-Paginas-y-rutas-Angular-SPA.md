---
title: Páginas y rutas Angular SPA
permalink: paginas-y-rutas-angular-spa
date: 2018-08-24 12:19:14
tags:  
- Angular
- SPA
- Routing
- Tutorial
- Introducción
categories:
- [Tutorial, Angular] 
thumbnail: /css/images/angular-2_spa.png
---
![Tutorial Angular5 2-SPA](/images/tutorial-angular-2_spa.png)


Las **aplicaciones Angular 6 son conjuntos de páginas enrutadas** en el propio navegador. Son las conocidas *SPA, Single Page Applications*. Estas apps liberan al servidor de una parte del trabajo, reducen la cantidad de llamadas y mejoran la percepción de velocidad del usuario.

Seguimos usando el concepto de árbol, ahora como analogía de **las rutas y las vistas** asociadas. Algo que se consigue fácilmente con `@angular/router` **el enrutador de Angular**.

<!-- more -->

Partiendo de la aplicación tal cómo quedó en [Base para una aplicación Angular](../base-aplicacion-angular/). Al finalizar tendrás un SPA con vistas asociadas a sus rutas.

>Código asociado a este artículo en *GitHub*: [AcademiaBinaria/AutoBot/2-spa](https://github.com/AcademiaBinaria/autobot/tree/2-spa) 

# 1. Rutas

Al crear la aplicación hice uso del flag `routing true` en el comando de generación del *CLI*. Esto causó la aparición de no uno, sino dos módulos gemelos en la raíz de la aplicación. Has estudiado el `AppModule` verdadero módulo raíz, y ahora verás su gemelo: el **módulo de enrutado** `AppRoutingModule` .

## 1.1 RouterModule

El módulo `AppRoutingModule` cumple dos funciones. Por un lado **importa al `RouterModule`** de Angular, el cual contiene toda la lógica necesaria para enrutar en el navegador. Por otro lado, permite la **definición de rutas** en el array `Routes[]`. 

```typescript
import { Routes, RouterModule } from "@angular/router";
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
    path: 'car',
    loadChildren: './car/car.module#CarModule'
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

El array de rutas recibe **objetos ruta** con propiedades de configuración. 

La primera es `path:` en la que se especifica **la dirección** que resuelve, en este caso la ruta vacía o raíz del árbol de rutas. Las otras son opcionales y las veremos poco a poco.

Empecemos casi por el final y de paso hagamos algo útil para no perdernos. La propiedad `component` es fundamental pues indica **el componente** que se debe mostrar cuando esta ruta se active. pPero hay más...

### 1.1.1 Router Redirect

La configuración de rutas no sólo permite asignar componentes a las direcciones. También se pueden hacer **redirecciones de unas direcciones a otras**. Y por supuesto puede haber **rutas no contempladas o errores** por parte del usuario, los infames `404 Not Found`.

> En este caso cuando se escriba la ruta `/not-foud` se mostrará un componente, el `NotFoundComponent`, cuyo contenido indicará al usaurio que se ha perdido. Claro que nadie va voluntariamente a esa ruta. Mediante el `path: '**'` le indico que ante cualquier ruta no contemplada anteriormente se ejecute el comando `redirectTo: 'not-found'`, el cual nos lleva a una ruta conocida con un mensaje bien conocido. Page Not Found.

Pero, ¿qué pasará con el componente activado? ¿dónde se cargará?. Presentamos a `<router-outlet>`.

## 1.2 Router Outlet

La idea general de **una SPA es tener una única página que cargue dinámicamente otras vistas**. Normalmente la página contenedora mantiene el menú de navegación, el pie de página y otras áreas comunes. Y deja un espacio para la carga dinámica. Para ello necesitamos saber **qué componente cargar y dónde mostrarlo**. De esto último se ocupa la etiqueta ` <router-outlet></router-outlet>`.

En el `main.component.ts` había un contenido *hard-coded*. Para hacerlo dinámico se sustituye por por el elemento de angular `<router-outlet></router-outlet>`. El cual inyectará dinámicamente el componente que le corresponda según la ruta activa. Queda algo así:

```typescript
    selector: 'app-main',
    template: `
    <main class="section">
      <div class="container">
        <router-outlet></router-outlet>
      </div>
    </main>
    `,
    styles: []
```

## 1.4 Router Link

Los enlaces web tradicionalmente se han resuelto con elementos `<a href=""></a>` dónde en su atributo `href` se asociaba la dirección a la cuál navegar ante el click del usuario. **En Angular los enlaces se declaran con un atributo** especial llamado `routerLink`. Este atributo **se compila dando lugar al `href`** oportuno.

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

>Por ahora la funcionalidad de `routerLink` no mejora en nada a `href`. Pero lo hará. Mientras tanto familiarízate con su sintaxis y... asegúrate de importar `RouterModule` en los módulos en los que lo vayas a usar. 


# 2 Lazy Loading
Las *webs SPA* se crearon por una razón que casi acaba con ellas: **la velocidad**. Al realizar el enrutado en el cliente y querer evitar todos los viajes posibles hasta el servidor, se cargó a la única página web con todo el peso de la aplicación. Lo cual la hizo terriblemente lenta en la primera visita de cada usuario. 

El **impacto de la primera visita** en una aplicación de intranet no suele ser un problema grave. Pero en internet esa visita puede ser la primera y la última. La solución viene de mano del concepto de *lazy loading* o carga perezosa. Consiste en diferir la carga de la lógica asociada a una dirección hasta el momento en que sea activada dicha ruta. De esa forma, **una página no visitada es una página que no pesa**. Y la carga inicial se hace mucho más liviana.

En Angular 6 el *lazy loading* es tan sencillo que ya se recomienda implementarlo por defecto. Para hacerlo conoceremos más comandos del `Router` y algunas herramientas de compilación usadas por el *Angular CLI*.

## 2.1 Webpack y los bundles por ruta

 Hay que saber que el *Angular CLI* usa internamente la herramienta de empaquetado *webpack*. La cual recorre el código *TypeScript* buscando `imports` y empaquetando su contendo en sacos o *bundles*. 
 
 Objetivo: adelgazar el peso del *bundle* principal, el `main.js`.Para conseguirlo hay que configurar las rutas de forma que no sea necesario importar los componentes a mostrar tal como se ha hecho con el `NotFoundComponent`. De hacerlo así con todos, *webpack* empaquetaría esos componentes como algo necesario... y por tanto serían enviados al navegador en el *bundle* principal sin que sea seguro su uso. 
 
 La solución que ofrecen el *cli* y *webpack* consiste en **delegar la asignación del componente a otro módulo, pero sin importarlo** hasta que su ruta principal se active.

He creado unas vistas para ser usada en las direcciones `/` u  `/about`. Los componentes asociados se llaman `HomeComponent` u `AboutComponent`. Se han **declarado pero no exportado** en sus repectivos módulos `HomeModule` y `AboutModule`. No es necesario exportarlos porque no será reclamdao directamente por nuestro código.

```bash
ng g m home --routing true
ng g c home/home
ng g m about --routing true
ng g c about/about
```

Estos módulo no debe ser importado por el `AppModule`. Simplemente debe usarse su ruta relativa en el módulo de enrutado `AppRoutingModule` como un valor especial. Fíjate que **la dirección del fichero es una cadena de texto** asignada a una nueva propiedad de objeto *route*, la propiedad `loadChildren:""`. No se está produciendo ninguna importación en *TypeScript* como ocurre con el componente `NotFoundComponent`.

Con esta información *webpack* va a generar un *bundle* específico para cada módulo. Si durante la ejecución se activa la ruta `/` (muy probable porque es la ruta raíz) o la ruta `/about` entonces se descarga ese paquete concreto y se ejecuta su contenido. Mientras tanto, se queda almacenado en el servidor. 

> Esto hace que la aplicación de Angular pese menos y responda antes, mejorando el tiempo de pintado inicial. La combinación de estas y otras técnicas que veremos en este tutorial sacarán el mejor rendimiento posible a tu aplicación Angular 6.

## 2.2 El enrutador delegado
Ya sabemos que hasta que no se active la ruta `/` o la `/about` no hay que hacer nada. Pero si se activa, entonces se descarga un *bundle* que contiene un módulo y los componentes necesarios. Sólo falta escoger dentro de ese módulo el componente que se asignará a la ruta.

Para eso al crear el módulo de operaciones usé el *flag* `routing true`. Esto hace que se genere un segundo módulo de enrutado. El `HomeRoutingModule` y el `AboutRoutingModule` son  prácticamente idénticos al enrutador raíz. 

> Digamos que son **enrutadores subordinados** al primero. Sólo se llega aquí si en la ruta principal se ha navegado a una dirección concreta. Se hace notar esa distinción durante el proceso de importación del módulo de Angular `RouterModule`. En el caso principal se pone `imports: [RouterModule.forRoot(routes)]` y en todos los demás `imports: [RouterModule.forChild(routes)]`.

A nivel subordinado, la dirección `path: ""` se agrega al `path: ""` de su enrutador padre. Cuidado, es un error común repetir el *path* a nivel hijo.

La ventaja real de este segundo enrutador es que irá empaquetado en el mismo *bundle* que el módulo de negocio y sus componentes. Aquí sí que asignaremos un componente concreto: el `HomeComponent` o el `AboutComponent`. Por ejemplo el fichero `home-routing.module.ts` quedará más o menos así:

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

# 3 Parámetros

Las rutas vistas hasta ahora se consideran estáticas pues se han definido usando constantes. Es muy habitual tener **páginas con la misma estructura pero distintos contenidos**. Un blog con sus posts, una tienda con sus productos, o un garaje con sus coches... hay miles de ejemplos así.

Ese tipo de direcciones se consideran paramétricas, tienen unos segmentos estáticos y otros dinámicos. Estos últimos se definen con parámetros, algo así como **variables dentro de la cadena de la ruta**. Su sintaxis obliga a precederlas de dos puntos. Por ejemplo `countries/:country/cities/:city` resolvería rutas como *countries/usa/cities/new-york* o *countries/france/cities/paris*. Rellenando los parámetros `:country` y `:city` con los valores necesarios.  

Volvamos a los coches. Vamos a crear rutas como */car/model-s* o */car/roadster*. Para ello necesitamos el segmento principal */car* y delegar su uso al `CarModule`.

```bash
ng g m car --routing true
ng g c car/car
```
El `car-routing.module.ts` tendrá este contenido.

```typescript
import { CarComponent } from './car/car.component';
const routes: Routes = [
  {
    path: ':carId',
    component: CarComponent
  }
];
```

Esta configuración resuelve las rutas `car/cualquier-cosa`. Y las carga con el componente `CarComponent`. El segundo segmento se almacenará como parámetro y será recogido con el nombre `carId`.

Para forzar los enlaces he creado un listado en el `HomeComponent`. La parte interesante de su *html* es:

```html
<ul class="menu-list">
  <li><a [routerLink]="['/car', 'Model S']">Model S</a></li>
  <li><a [routerLink]="['/car', 'Model X']">Model X</a></li>
  <li><a [routerLink]="['/car', 'Model 3']">Model 3</a></li>
  <li><a [routerLink]="['/car', 'Roadster']">Roadster</a></li>
</ul>
```

Aún más interesante es el componente que muestra cada elemento de la lista, el `CarComponent`. En este caso fíjate cómo accede a la ruta, obtiene el valor del parámetro y lo usa para mostrarlo en la web. 

Contenido del fichero `car.component.ts` relacionado con la obtención del parámetro de la ruta activa:

```typescript
import { Component, OnInit } from "@angular/core";
import { ActivatedRoute } from "@angular/router";
@Component({
  selector: "app-car",
  template: ` 
    <div class="card-header-title">
      {{ carId }}
    </div>`
})
export class CarComponent implements OnInit {
  public carId;
  constructor(private route: ActivatedRoute) {}
  ngOnInit() {
    this.carId = this.route.snapshot.params['carId'];
  }
}
```

## 3.1 ActivatedRoute

El framework *Angular* 6 trae muchas librerías para facilitar la vida al programador. Sólo hay que saber dónde están y cómo pedirlas. Para ello volvemos a la tecnología escogida,  *TypeScript*, que permite las **importaciones y la inyección de dependencias**.

La instrucción `import { ActivatedRoute } from "@angular/router";` pone a disposición del programador el código donde está definida la clase `ActivatedRoute`. Pero no se instancia directamente, en su lugar, se usa como un argumento del constructor de la clase del componente. Ese constructor es invocado por *Angular*, y dinámicamente el propio framework sabe cómo rellenar los argumentos que se pidan en los constructores. Es decir, sabe cómo inyectar instancias en las que dependencias declaradas.

Una vez que han **inyectan las dependencias en el constructor** ya están listas para ser usadas como propiedades de la clase. *Mágia del TypeScript*. En concreto `this.route` da acceso a métodos y propiedades para trabajar con la ruta activa y poder leer sus parámetros.

## 3.2 Eventos e interfaces en TypeScript

El lenguaje *TypeScript* como superconjunto de *JavaScript* aporta técnicas de P.O.O. bien conocidas en lenguajes como *Java* o *C#*. Por ejemplo **la herencia y los interfaces**. Los diseñadores de *Angular* decidieron usar interfaces para implementar el **ciclo de vida de los componentes**. En lugar de lanzar eventos a los que subscribirse, te piden que implementes métodos de distintas interfaces. Esos métodos serán llamados cuando corresponda, como si fuesen subscripciones a eventos.

En este caso la *interfaz* `OnInit` obliga a implementar el método `ngOnInit()` el cual será invocado lo antes posible pero tras la completa construcción del componente. Asegurando así que el código que se ejecute en ese método tenga acceso a un componente completo y totalmente listo. Esta es la manera recomendada para ejecutar código de inicialización de los componentes, dejando los constructores vacíos y sólo dedicados a declarar dependencias.

# 4 Rutas anidadas

Cuando las intefaces se complican, es habitual que las aplicaciones dsiponga de menús de navegación a distintos niveles. Dentro de una misma página podemos quere ver distinto contenido y admas reflejarlo en la *url*. Para resolver esta situación en Angular 6 disponemos de la técnica de las *nested routes*.

> De una manera un tanto forzada la he incluído en la página `/about`. La cual disponen de su propio menú de navegación, y lo que es más importante, su propio `<router-outlet></router-outlet>`.

Para empezar veamos como queda el *html* del `about.component.ts`. Vamos a dotarlo de dos rutas nuevas `/about/links` y `/about/info`. Cada una mostrará contenido en un componente adecuadamente insertado en el `<router-outlet></router-outlet>` local.

```html
<nav class="navbar" role="navigation">
  <div class="navbar-menu is-active">
    <div class="navbar-start">
      <a class="navbar-item"  [routerLink]="['./links']"> Links</a>
      <a class="navbar-item"  [routerLink]="['./info']"> Info</a>
    </div>
  </div>
</nav>
<router-outlet></router-outlet>
```

Para que funcione empezamos por crear los dos compoentes `LinksComponent` e `InfoComponent` de forma rutinaria. Y los asignamos en el `about-routing.module.ts` como subordinados a la ruta principal con el comando `children:[]`.

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

La combinación de `children, loadChild, component, redirectTo` ... asociadas a `path` es lo que te dará toda la potencia para configurar tu aplcación y responder a cualquier *url* desde la misma y única página `index.html`. 

Con esto tendrás una aplicación SPA en *Angular*. Sigue esta serie para añadirle [Formularios, tablas y modelos de datos en Angular](../formularios-tablas-y-modelos-de-datos-en-angular/) mientras aprendes a programar con Angular6.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>
