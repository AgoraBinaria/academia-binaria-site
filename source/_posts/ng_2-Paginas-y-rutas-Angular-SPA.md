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
![Tutorial Angular5 2-SPA](/images/tutorial-angular-5_2_SPA.png)


Las **aplicaciones Angular son conjuntos de páginas enrutadas** en el propio navegador. Son las conocidas *SPA, Single Page Applications*. Estas apps liberan al servidor de una parte del trabajo, reducen la cantidad de llamadas y mejoran la percepción de velocidad del usuario.

Seguimos usando el concepto de árbol, ahora como analogía de **las rutas y las vistas** asociadas. Algo que se consigue fácilmente con `@angular/router` **el enrutador de Angular**.

<!-- more -->

Partiendo de la aplicación tal cómo quedó en [Base para una aplicación Angular](../base-aplicacion-angular/). Al finalizar tendrás un SPA con vistas asociadas a sus rutas.

>Código asociado a este artículo en *GitHub*: [AcademiaBinaria/angular5/2-spa](https://github.com/AcademiaBinaria/angular5/tree/master/2-spa/cash-flow) 


# 1. Rutas
Al crear la aplicación hice uso del flag `routing true` en el comando de generación del *CLI*. Esto causó la aparición de no uno, sino dos módulos gemelos en la raíz de la aplicación. Has estudiado el `AppModule` verdadero módulo raíz, y ahora verás su gemelo: el **módulo de enrutado** 'AppRoutingModule'.

## 1.1 RouterModule
Este módulo cumple dos funciones. Por un lado **importa al `RouterModule`** que contiene toda la lógica necesaria para enrutar en el navegador. Por otro lado, permite la **definición de rutas** en el array `Routes[]`. 

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

El array de rutas recibe **objetos con propiedades** de configuración. La primera es `path:` en la que se especifica **la dirección** que resuelve, en este caso la ruta vacía o raíz del árbol de rutas. Otra propiedad fundamental es `component` la cual indica **el componente** que se debe mostrar cuando esta ruta se active.

En este caso he aprovechado el componente `HomeComponent` para asociarlo a la ruta raíz. Pero, ¿qué pasará con dicho componente? ¿dónde se cargará?. Presentamos a `<router-outlet>`.

## 1.2 Router Outlet 
La idea general de **una SPA es tener una única página que cargue dinámicamente otras vistas**. Normalmente la página contenedora mantiene el menú de navegación, el pie de página y otras áreas comunes. Y deja un espacio para la carga dinámica. Para ello necesitamos saber **qué componente cargar y dónde mostrarlo**. De esto último se ocupa la etiqueta ` <router-outlet></router-outlet>`.

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
La configuración de rutas no sólo permite asignar componentes a las direcciones. También se pueden hacer **redirecciones de unas direcciones a otras**. Y por supuesto puede haber **rutas no contempladas o errores** por parte del usuario, los infames `404 Not Found`.

Un ejemplo de ambas situaciones sería configurar nuestras rutas de forma que toda ruta desconocida nos lleve a otra general que muestre un mensaje predeterminado. Para hacerlo genera un módulo y un componente llamados `NotFound` con los siguientes comandos:

```shell
ng g m views/not-found
ng g c views/not-found/not-found --export --flat
```
Vuelve al módulo de enrutado, ahora conocido como `app.routing.ts`, y agrega dos nuevas entradas al array `routes`.

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

>Por ahora la funcionalidad de `routerLink` no mejora en nada a `href`. Pero lo hará. Mientras tanto familiarízate con su sintaxis y... asegúrate de importar `RouterModule` en `not-found.module.ts`. Puedes ver los cambios realizados hasta ahora en [este *commit*](https://github.com/AcademiaBinaria/angular5/commit/464a5f0fd5f8975157793b0a3c13d7f61f890fa5) 


# 2 Lazy Loading
Las *webs SPA* se crearon por una razón que casi acaba con ellas: **la velocidad**. Al realizar el enrutado en el cliente y querer evitar todos los viajes posibles hasta el servidor, se cargó a la única página web con todo el peso de la aplicación. Lo cual la hizo terriblemente lenta en la primera visita de cada usuario. 

El **impacto de la primera visita** en una aplicación de intranet no suele ser un problema grave. Pero en internet esa visita puede ser la primera y la última. La solución viene de mano del concepto de *lazy loading* o carga perezosa. Consiste en diferir la carga de la lógica asociada a una dirección hasta el momento en que sea activada dicha ruta. De esa forma, **una página no visitada es una página que no pesa**. Y la carga inicial se hace mucho más liviana.

## 2.1 Webpack y los bundles por ruta
Objetivo: adelgazar el peso del *bundle* principal. Para conseguirlo hay que configurar las rutas de forma que no sea necesario importar los componentes a mostrar. De otro modo *webpack* empaquetaría ese componente como algo necesario... y por tanto sería enviado al navegador en el *bundle* principal sin que sea seguro su uso. La solución que ofrecen el *cli* y *webpack* consiste en **delegar la asignación del componente a otro módulo, pero sin importarlo**.

He creado un una nueva vista para ser usada en una nueva dirección llamada `/operations`. El componente se llama `OperationsComponent` y se ha **declarado pero no exportado** en el módulo `OperationsModule`. 

```shell
ng g m views/operations --routing true
ng g c views/operations/operations --flat
```

Este módulo no debe ser importado por el `AppModule`. Simplemente debe usarse su ruta relativa en el módulo de enrutado `AppRoutingModule` como un valor especial. Fíjate que **la dirección del fichero es una cadena de texto** asignada a una nueva propiedad de objeto route, la propiedad `loadChildren:""`. No se está produciendo ninguna importación en *TypeScript* como ocurre con los componentes `HomeComponent` y  `NotFoundComponent`.

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
Con esta información *webpack* va a generar un *bundle* específico para este módulo. Si durante la ejecución se activa la ruta `/operations` entonces descarga ese paquete y ejecuta su contenido. Mientras tanto, se queda almacenado en el servidor.

## 2.2 El enrutador delegado
Ya sabemos que hasta que no se active la ruta `/operations` no hay que hacer nada. Pero si se activa, entonces se descarga un *bundle* que contiene un módulo y los componentes necesarios. Sólo falta escoger dentro de ese módulo el componente que se asignará a la ruta.

Para eso al crear el módulo de operaciones usé el *flag* `routing true`. Esto hace que se genere un segundo módulo de enrutado. El `OperationsRoutingModule` prácticamente idéntico al enrutador raíz. Digamos que es **un enrutador subordinado** al primero. Sólo se llega aquí si en la ruta principal se ha navegado a una dirección concreta. A este nivel la dirección `path: ""` se agrega al `path: "operations"` de su enrutador padre.

La ventaja real de este segundo enrutador es que irá empaquetado en el mismo *bundle* que el módulo de negocio y sus componentes. Aquí sí que asignaremos un componente concreto: el `OperationsComponent`. Dejando el fichero `operations.routing.ts` más o menos así:

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
Las rutas vistas hasta ahora se consideran estáticas pues se han definido usando constantes. Es muy habitual tener **páginas con la misma estructura pero distintos contenidos**. Un blog con sus posts, una tienda con sus productos... hay miles de ejemplos así.

Ese tipo de direcciones se consideran paramétricas, tienen unos segmentos estáticos y otros dinámicos. Estos últimos se definen con parámetros, algo así como **variables dentro de la cadena de la ruta**. Su sintaxis obliga a precederlas de dos puntos. Por ejemplo `countries/:country/cities/:city` resolvería rutas como *countries/usa/cities/new-york* o *countries/france/cities/paris*. Rellenando los parámetros `:country` y `:city` con los valores necesarios.  

```typescript
import { OperationsComponent } from "./operations.component";
const routes: Routes = [
  {
    path: "",
    component: OperationsComponent
  },
  {
    path: ":id",
    component: ItemComponent
  }
];
```
Esta configuración resuelve las rutas `operations` y `operations/cualquier-cosa`. En la primera carga `OperationsComponent` y en los demás casos el `ItemComponent`.

> En la práctica que nos ocupa lo usaremos para ver el detalle de las operaciones económicas realizadas. Como por ahora no tenemos, he puesto de ejemplo algunos números bien conocidos.

Para forzar los enlaces he creado un componente a modo de listado llamado `ListComponent`. La parte interesante de su *html* es:

```html
<ul>
  <li><a routerLink="/operations/271">Number e</a></li>
  <li><a routerLink="/operations/314">Pi</a></li>
  <li><a routerLink="/operations/667">Gravitational Constant</a></li>
</ul>
```

Aún más interesante es el componente que muestra cada elemento de la lista, el `ItemComponent`. En este caso fíjate cómo accede a la ruta, obtiene el valor del parámetro y lo usa para mostrarlo en la web. 

Contenido del fichero `item.component.ts`:
```typescript
import { Component, OnInit } from "@angular/core";
import { ActivatedRoute } from "@angular/router";
@Component({
  selector: "cf-item",
  template: ` <h3>{{ _id }}</h3>`
})
export class ItemComponent implements OnInit {
  _id: any;
  constructor(private route: ActivatedRoute) {}
  ngOnInit() {
    this._id = this.route.snapshot.params["id"];
  }
}
```

# 3.1 ActivatedRoute
El framework *Angular* trae muchas librerías para facilitar la vida al programador. Sólo hay que saber dónde están y cómo pedirlas. Para ello volvemos a la tecnología escogida,  *TypeScript*, que permite las **importaciones y la inyección de dependencias**.

La instrucción `import { ActivatedRoute } from "@angular/router";` pone a disposición del programdor el código donde está definida la clase `ActivatedRoute`, pero no se instancia directamente. En su lugar, se usa como un argumento del constructor de la clase del componente. Ese constructor es invocado por *Angular*, el cual sabe cómo rellenar los argumentos que le pido. Es decir, sabe cómo inyectar instancias de las que dependo.

Una vez que me **inyectan las dependencias en el constructor** ya están listas para ser usadas como propiedades de la clase. *Mágia del TypeScript*. En concreto `this.route` me da acceso a métodos y propiedades para trabajar con la ruta activa y poder leer sus parámetros.

# 3.2 Eventos e interfaces en TypeScript
El lenguaje *TypeScript* como superconjunto de *JavaScript* aporta técnicas de P.O.O. bien conocidas en lenguajes como *Java* o *C#*. Por ejemplo **la herencia y los interfaces**. Los diseñadores de *Angular* decidieron usar interfaces para implementar el **ciclo de vida de los componentes**. En lugar de lanzar eventos a los que subscribirse, te piden que implementes métodos de distintas interfaces. Esos métodos serán llamados cuando corresponda, como si fuesen subscripciones a eventos.

En este caso la *interfaz* `OnInit` obliga a implementar el método `ngOnInit()` el cual será invocado lo antes posible pero tras la completa construcción del componente. Asegurando así que el código que se ejecute en ese método tenga acceso a un componente completo y totalmente listo. 

> En [este *commit*](https://github.com/AcademiaBinaria/angular5/commit/0afd90532bcce9f95bca130838eb11d5a4a3b491) puedes ver los cambios necesarios para incluir parámetros en la aplicación.

Con esto tendrás una aplicación SPA en *Angular*. Sigue esta serie para añadirle funcionalidad mientras aprendes a programar con Angular5.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>
