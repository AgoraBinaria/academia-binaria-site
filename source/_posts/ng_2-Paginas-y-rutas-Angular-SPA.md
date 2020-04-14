---
title: Páginas y rutas Angular SPA
permalink: paginas-y-rutas-angular-spa
date: 2020-04-14 13:41:14
tags:
  - Angular
  - SPA
  - Routing
  - Tutorial
  - Introducción
  - Angular9
  - Angular2
categories:
  - [Tutorial, Angular]
thumbnail: /css/images/angular-2_spa.png
---

![paginas-y-rutas-angular-spa](/images/tutorial-angular-2_spa.png)

Las **aplicaciones Angular 9 son conjuntos de páginas enrutadas** en el propio navegador. Son las conocidas _SPA, Single Page Applications_. Estas apps liberan al servidor de una parte del trabajo reduciendo la cantidad de llamadas y mejorando la percepción de velocidad del usuario.

En este tutorial aprenderás a crear una Angular SPA fácilmente usando `@angular/router`, **el enrutador de Angular**.

<!-- more -->

Partiendo de la aplicación tal como quedó en [Base para una aplicación Angular](../base-aplicacion-angular/). Seguimos usando el concepto de árbol, ahora como analogía de **las rutas y las vistas** asociadas. Al finalizar tendrás una angular SPA con vistas asociadas a sus rutas.

> Código asociado a este tutorial en _GitHub_: [AcademiaBinaria/angular-basic](https://github.com/AcademiaBinaria/angular-basic/)

# 1. Rutas

Es raro que una aplicación web resuelva o exponga toda su funcionalidad e información e una única vista. Lo habitual es que **se desplieguen múltiples páginas en distintas direrecciones**. Hasta hace unos años la única opción era que el servidor procesase dicha ruta y remitiese el contenido listo para visualizar en el navegador.

Esto significa mucho trabajo para el servidor, mucho contenido para la red y poca responsabilidad para los navegadores. Para las aplicaciones empresariales parece razonable distribuir esa carga, y **que sea el navegador el que prepare la vista** ejecutando instrucciones y solicitando datos.

Y una de las responsabilidades de las que se hará cargo es la de **procesar las rutas y determinar cual será la vista** que se deba mostrar en cada dirección. Veamos cómo lo resuelve Angular.

Al crear la aplicación hice uso del flag `routing` en el comando de generación del _CLI_.

Recordemos:

```terminal
ng new angular-basic --routing
```

Esto causó la aparición de no uno, sino dos módulos gemelos en la raíz de la aplicación. Has estudiado el `AppModule` verdadero módulo raíz, y ahora verás en profundidad a su gemelo: el
**módulo de enrutado** `AppRoutingModule` y el uso que hace del `RouterModule`.

## 1.1 RouterModule

Con lo que hemos aprendido sobre módulos y sus dependencias podemos entender que
`AppRoutingModule` importa, configura y exportar al `RouterModule`. Y que a su vez `AppModule` al importar a `AppRoutingModule` dispone de todo lo necesario para realizar el enrutado.

La ruta de dependencias de módulos queda tal que así:

```
RouterModule -> AppRoutingModule -> AppModule
```

Hasta ahora los módulos habían sido meros contendores. Algo similar a los espacios de nombres. Pero al ser clases puede tener código y por tanto exponer funcionalidad. De hecho, el `RouterModule` expone un par de métodos de configuración. Se llaman `.forRoot(routes:Routes)` y `.forChild(routes:Routes)` y se usan a nivel raíz o todas las demás situaciones respectivamente.

Ambos reciben una estructura que mantiene un array de rutas y las instrucciones a ejecutar cuando dichas rutas se activen. Las rutas pueden ser estáticas o usar comodines. Las acciones pueden ser de elección de componente para la vista, diferir el trabajo a otro módulo o redirigir al usuario a otra ruta.

### Módulos componentes y rutas

Vemos un primer ejemplo para el enrutador a nivel raíz. Partimos del par módulo-componente para la página _Home_. Hasta ahora se veía en la aplicación porque estaba incrustado a mano en medio del _layout_ principal.

Lo que haremos a continuación es **asignar este componente a una ruta**, y que sólo se vea cuando le toque a dicha ruta. Claro que en este caso es la ruta vacía, y por ahora es la única así que no cambiará gran cosa. Pero, de este modo Angular sabrá que mostrar en cada ruta para cuando hay más.

```typescript
import { Routes, RouterModule } from '@angular/router';
import { HomeComponent } from './home/home.component';

const routes: Routes = [
  {
    path: '',
    component: HomeComponent,
  }
];
@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule {}
```

## 1.2 Router Outlet

El caso es que necesitamos mostrar un componente u otro en función de una ruta. Por tanto habrá que eliminar la referencia explícita a `<ab-home>` y confiar en algo que ya estaba presente pero que no habíamos estudiado, el `<router-outlet>`.

Este es un componente que viene con el `RouterModule` y actúa como un **contenedor dinámico**, incrustando el componente adecuado apara cada ruta. El contenido de `main.component.ts`, ahora será dinámico

```html
<main>
  <p>
    Fork this <a href="https://github.com/AcademiaBinaria/angular-basic">Repository</a>
  </p>
  <router-outlet></router-outlet>
</main>
```

## 1.3 Router Link

Otra novedad que podemos, y debemos, empezar a usar es la directiva `routerLink`. **Una directiva es una extensión del HTML propia de Angular.** Se emplea como si fuese un atributo de cualquier elemento y durante la compilación genera el código estándar necesario para que lo entiendan los navegadores.

En concreto esta directiva, que también viene en el módulo `routerModule`, se usa en sustitución del atributo estándar `href`. Inicialmente nos basta con saber que instruye al navegador para que no solicite la ruta al servidor, sino que el propio código local de javaScript se encargará de procesarla.

Así, por ejemplo en el único y sencillo componente compartido del que disponemos, decidimos usarla para que las idas y venidas entre nuestras rutas no requieran de recarga en el servidor.

En el `src\app\shared\go-home\go-home.component.html`
```html
<a routerLink=""> Go home 🏠</a>
```

> Recuerda, `routerLink` es una _Directiva_
>>Como un atributo, pero con superpoderes

Por ahora, _simplemente_ mantiene la gestión de las rutas en el lado del navegador.

# 2 Lazy Loading

Tal com hemos procedido para la ruta vacía, podríamos continuar con todas las demás. Por ejemplo una ruta muy común sería la típica _Acerca de_ o dicho en modo _url_: `/about`. Lo que haríamos sería generarle un módulo con un un componente y luego asignar dicho componente a la tabla de rutas con algo así:

```typescript
import { Routes, RouterModule } from '@angular/router';
import { HomeComponent } from './home/home.component';
import { AboutComponent } from './about/about.component';

const routes: Routes = [
  {
    path: '',
    component: HomeComponent,
  },
  {
    path: 'about',
    component: AboutUsComponent,
  }
];
@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule {}
```

Podríamos, pero no lo haremos. ¿Por qué? Pues por una **cuestión de pesos y velocidad**. Tal como se ve en el código, para poder asignar un componente a una ruta antes tenemos que haberlo importado. Y eso significa que parará a formar parte del código que se _transpile_, empaquete y envíe al navegador.

Es decir, el sufrido usuario se va a **descargar la definición de los componentes antes de visitarlos**. Es más, se descargará componentes de rutas que quizá nunca visite. A esta técnica se la conoce como _eager loading_ y, en general y hablando así a la bruto, debemos evitarla en favor de otra conocida como _lazy loading_.


## 2.1 Webpack y los bundles por ruta

Para implementarla se necesitan un par de cooperantes. En particular y sobre todo el empaquetador _Webpack_.

El objetivo es **diferir la descarga de las rutas no visitadas** y para ello querremos empaquetar cada ruta en un _bundle_. Esto requiere al menos un módulo por ruta y adoptar un convenio especial para que  _webpack_ inicie nuevos empaquetados en múltiples puntos.

### Crear los componentes en módulos con enrutado

Por complejo que suene en la práctica es muy sencillo. Basta con usar el comando adecuado del CLI. Por ejemplo para el caso del _Acerca de_ emplearíamos una instrucción como esta:

```bash
ng g m about --route=about -m app-routing.module.ts
```
No es más que la generación de un nuevo módulo pero con el _flag_ `--route=` que le indica al CLI que debe tratarlo como una nueva ruta. Este súper comando genera dos módulos, un componente y además los registra automáticamente. Veamos el resultado:

En el módulo de enrutado raíz tenemos un nuevo camino, pero con una sintaxis distinta.

```typescript
const routes: Routes = [
  {
    path: '',
    component: HomeComponent,
  },
  {
    path: 'about',
    loadChildren: () => import('./about/about.module').then(m => m.AboutModule),
  },
];
```

Lo que dice es que cuando se active la ruta `about` entonces se le transfiera el control a otro módulo mediante una instrucción asíncrona. De esta forma ase consiguen dos cosas: por un lado al no usar ningún componente explícito no hay que importarlo; por otro lado la descarga del módulo que resuelva el problema se ejecutará en segundo plano y sólo si el usuario visita la ruta.

## 2.2 El enrutador delegado

Claro que sólo hemos visto la mitad de la película. La instrucción `loadChildren` delega el enrutado en otro módulo; el `AboutModule` que fue creado por el cli. Dicho módulo depende a su vez de otro de enrutamiento local, el `AboutRoutingModule`.

Este módulo de enrutamiento es similar al ya conocido `AppRoutingModule`, pero se activa y por tanto actúa, a partir de una ruta ya procesada por su padre. Su contenido es similar a esto:


```typescript
import { AboutComponent } from './about.component';

const routes: Routes = [{ path: '', component: AboutComponent }];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule],
})
export class AboutRoutingModule {}
```

Tres cosas llaman la atención. Para empezar la ruta está vacía de nuevo. Pero eso es por que se concatena a la anteriormente evaluada. Es decir, el segmento `about` está ya descontado. Segundo, a este nivel volvemos a indicar un componente concreto y por tanto necesitamos importarlo. Por último y hablando de importaciones, el `RouterModule` se configura ahora como una rama hija del árbol de rutas principal. Lo hace con el método `.forChild(routes: Routes)`.

> Comprueba en ejecución cómo se descargan los _bundles_ según navegas.

### La navegación lazy permite la descarga diferida al navegar por las rutas.

# 3. Rutas anidadas

Hay muchas situaciones que **por cuestiones de usabilidad anidamos navegaciones**. Por ejemplo una tienda online, te permite escoger categorías, y después vistas distintas de sus productos como listados o fichas. En las aplicaciones de gestión es frecuente encontrarse con estructuras tipo tab o menús de actuación parciales.

Estas situaciones se resuelven la tecnología denominada _nested routes_ y requiere del conocimiento de una nueva propiedad de las rutas.

## 3.1 Children

Antes de nada supongamos que en la página about queremos mostrar dos categorías de información. Por un lado enlaces de interés sobre esta aplicación y por otro una información básica sobre la misma.

Crearíamos por tanto un par de componentes como estos.

```bash
ng g c about/about/links
ng g c about/about/info
```

Pero, en lugar de asignarles ya un camino específico a cada uno, lo que haremos será incrustarlos como hijos del componente `AboutComponent`. Para ello escribimos algo así en `about-routing.module.ts`

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

Esto es sólo el primer paso.

## 3.2 RouterOutlet anidado

Para que los hijos acaben apareciendo dónde deben, hay que usar un segundo `<router-outlet>`. Esta vez dentro de la vista del componente padre. En `AboutComponent` :

```html
<h3>About us</h3>
<header>
  <p>
    <a routerLink="links"
       class="button"> Tutorial Links </ab-home>
  </p>
  <p>
    <a routerLink="info"
       class="button"> More Info </a>
  </p>
</header>
<router-outlet></router-outlet>
```

Ahora este componente actúa como una micro aplicación; con su menú y su espacio de carga dinámica.

# 4 Rutas paramétricas

Si hay algo que poco tardará en aparecer será la necesidad de presentar **información distinta pero con un formato similar**. De nuevo ejemplos archiconocidos serán una página para un producto, un artículo en un blog, una ficha de empleado o el seguimiento de un pedido.

En estas situaciones queremos que parte de la ruta identifique al elemento concreto que vamos a mostrar, y a ese identificador le llamaremos **parámetro**. Ojo, es similar pero no exactamente un `queryParameter`.

## 4.1 Variables en la ruta

Por ejemplo, supongamos una academia que quiere mostrar una lista de cursos y una página para cada uno. Para empezar creará un módulo enrutado como este:

```bash
ng g m courses --route=courses -m app-routing.module.ts
```

Entre otras cosas modificará el `AppRoutingModule` incrustando una nueva entrada como esta:

```typescript
{
  path: 'courses',
  loadChildren: () => import('./courses/courses.module').then(m => m.CoursesModule)
},
```

Pero nosotros después vamos a realizar un cambio en la gestión local, incorporando un nuevo segmento al camino. Le asignamos el valor `:slug`.

```typescript
const routes: Routes = [
  {
    path: ':slug',
    component: CoursesComponent
    }
];
```

En este caso el `:` indica que lo que viene no es un texto literal, si no **una variable**. Un parámetro en nuestro argot. El nombre es cosa del programador, el usuario nunca lo verá. En este caso me he decidido por usar el término _slug_ muy empleado para introducir títulos dentro de las url.

Ahora ya resuelve rutas como: _/courses/introduccion_ o _/courses/avanzado_

Otra cosa será qué hacer cuando esas rutas se activen.

## 4.2 ActivatedRoute

Entramos quizá en la parte más compleja, pero que como siempre es igual te la puedes tomar como una receta para todas tus aplicaciones.

Veamos antes el contenido del fichero `courses.component.ts` relacionado con la obtención del parámetro de la ruta activa:

```typescript
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';
import { environment } from '../../environments/environment';

export class CoursesComponent implements OnInit {
  course: any;
  constructor(route: ActivatedRoute) {
    route.params.subscribe(params => {
      const courseSlug = params.slug;
      this.course = environment.courses.find(c => c.slug === courseSlug);
    });
  }
  ngOnInit() {}
}
```

Por partes, que es la primera vez que vemos tanto código en este tutorial. Para empezar y ya que **los componentes son clases**, entonces podrán tener propiedades, métodos y constructores. Así que aparecen la propiedad `course:any` sin tipo concreto. Y el constructor que recibe una argumento de tipo `ActivatedRoute`

**Angular adopta y promueve el uso de varios patones de arquitectura de software**. Uno de ellos es la inyección de dependencias, a la que dedicamos un tema en este tutorial. Por ahora nos basta con saber  que el framework nos inyectará una instancia de la clase `ActivatedRoute` en la variable argumento `route`.

Después viene un código intimidante pero que también es siempre del mismo tipo. La dificultad radica en usar programación asíncrona, a la que también dedicamos más de un tema. Simplificando, lo que nos dice es que si nos suscribimos a él, entonces nos notificará los cambios en los parámetros para que hagamos uso de ellos.

En este caso, buscamos el curso solicitado en un sencillo array. Y después se lo mostramos al usuario con técnicas de presentación dinámica propias de Angular que también se ven más adelante en este curso.

```html
<h3>👨‍🎓 {{ course.title }}</h3>
<p>{{course.description}}</p>
<p>
  <a href="{{course.url}}"
     target="_blank">{{course.url}}</a>
</p>
<p>
  <ab-go-home></ab-go-home>
</p>
```

Enlazamos todo cambiando los `href` del par de anclas del `HomeComponent`. Ahora usamos el enrutamiento local mediante `routerLink`.

```html
<h2> Welcome 🏡 !</h2>
<nav>
  <p>
    <a routerLink="courses/introduccion">💻 Introducción</a>
  </p>
  <p>
    <a routerLink="courses/avanzado">💻 Avanzado</a>
  </p>
</nav>
```

# 5 Redirecciones

Hay situaciones en las que dada una ruta, queremos **enviar al usuario a otra página**. A veces por una simple decisión de renombrado de rutas. Otras quizá respondiendo a problemas o acciones inesperadas del usuario.

Por ejemplo, vamos ver un tratamiento genérico del caso _not found_. Para empezar crearemos una ruta específica para indicarle al usuario que la ruta que buscaba no existe.

```terminal
ng g m not-found --route=not-found -m app-routing.module.ts
```

Ya sabemos lo que ocurre. Un nuevo módulo y una ruta diferida a nivel raíz  `not-found`.

```typescript
{
  path: 'not-found',
  loadChildren: () => import('./not-found/not-found.module').then(m => m.NotFoundModule),
},
```

Que localmente se asigna al componente `NotFoundComponent`.

```typescript
{
  path: '',
  component: NotFoundComponent,
},
```


> Pero, nadie va voluntariamente a esa ruta

>> Sólo los que se pierden

Así que hay que obligarles. Para eso usamos un nuevo comando de la configuración de rutas, el `redirectTo`. Y lo asignamos a todas aquellas rutas desconocidas usando un el comodín `**`

```typescript
{
  path: '**',
  redirectTo: 'not-found'
}
```

Esta entrada especial debe situarse **al final del _array_ de las rutas conocidas**. Angular evalúa la ruta actual contra todas las disponibles de arriba a abajo. La primer que resuelva el _match_ gana.

El conjunto de rutas de nuestra aplicación a estas alturas queda como sigue:

```typescript
import { Routes, RouterModule } from '@angular/router';
import { HomeComponent } from './home/home.component';

const routes: Routes = [
  {
    path: '',
    component: HomeComponent,
  },
  {
    path: 'about',
    loadChildren: () => import('./about-us/about-us.module').then(m => m.AboutUsModule),
  },
  {
    path: 'courses',
    loadChildren: () => import('./courses/courses.module').then(m => m.CoursesModule)
  },
  {
    path: 'not-found',
    loadChildren: () => import('./not-found/not-found.module').then(m => m.NotFoundModule),
  },
  {
    path: '**',
    redirectTo: 'not-found',
  },
];
@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule {}
```

La parte vistosa es crear el contenido para el  `NotFoundComponent`.

```html
<h3>404</h3>
<p> 🧭 not-found works!</p>
<ab-go-home></ab-go-home>
```

Con esto tendrás una aplicación SPA en _Angular_. Sigue esta serie para añadirle [Formularios, tablas y modelos de datos en Angular](../formularios-tablas-y-modelos-de-datos-en-angular/) mientras aprendes a programar con Angular9.Todos esos detalles se tratan en el [curso básico online](https://www.trainingit.es/curso-angular-basico/?promo=angular.builders) que imparto con TrainingIT o a medida para tu empresa.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>
