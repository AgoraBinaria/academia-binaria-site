---
title: Base para una aplicación Angular
permalink: base-aplicacion-angular
date: 2018-08-17 11:02:46
tags:  
- Angular
- CLI
- Tutorial
- Introducción
- Angular6
- Angular7
- Angular2
categories:
- [Tutorial, Angular] 
thumbnail: /css/images/angular-1_base.png
---
![base-aplicacion-angular](/images/tutorial-angular-1_base.png)

Vamos a crear una **base sobre la que programar una aplicación Angular 6** profesional. Usaremos el *CLI* para generar una estructura sobre la que crecer. Será como una semilla para un desarrollo controlado. 
La idea de árbol se usa en muchas analogías informáticas. La emplearemos en dos conceptos básicos en Angular: **los módulos y los componentes**.

<!-- more -->

Partimos de la aplicación tal cómo la dejamos en el [Hola Mundo en Angular](../hola-angular-cli/). Al finalizar tendrás un esqueleto del que colgar módulos y componentes funcionales.

>Código asociado a este artículo en *GitHub*: [AcademiaBinaria/AutoBot/1-base](https://github.com/AcademiaBinaria/autobot/tree/1-base) 

# 1. Módulos

Los módulos son **contenedores dónde almacenar los componentes y servicios** de una aplicación. En Angular cada programa se puede ver como un árbol de módulos jerárquico. A partir de un módulo raíz se enlazan otros módulos en un proceso llamado importación.

## 1.1 Definición mediante decoradores

Antes de importar cualquier módulo hay que definirlo. En Angular 6 **los módulos de declaran como clases de TypeScript**. Estas clases, habitualmente vacías, son decoradas con una función especial. Es la función `@NgModule()` que recibe un objeto como único argumento. En las propiedades de ese objeto es dónde se configura el módulo. 

Mira el módulo `AppModule` original que genera el CLI en el fichero `app.module.ts`.

```typescript
@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

## 1.2 Importación de otros módulos

El módulo `App` también se conoce como **módulo raíz** porque de él surgen las demás ramas que conforman una aplicación. La asignación de los nodos hijos se realiza en la propiedad `imports:[]`, que es un array de punteros a otros módulos.

>En la situación original el módulo principal depende un módulo *custom* (el `CoreModule` que usarás más adelante) y de otro *del framework* para la presentación en el navegador (el `BrowserModule`).

### 1.2.1 Dos mundos paralelos: imports de Angular 6 e import de TypeScript

Si es la primera vez que ves código TypeScript te llamarán la atención las primeras líneas de cada fichero. En el `app.module.ts` son algo así:

```typescript
import { BrowserModule } from "@angular/platform-browser";
import { NgModule } from "@angular/core";
import { AppComponent } from "./app.component";
```

Estas **sentencias de importación son propias del lenguaje** y nada tienen que ver con Angular 6. En ellas se indica que este fichero importa el contenido de otros ficheros *TypeScript*. La importación se realiza en base a convenios personalizables. Si empieza con `./` entonces se busca a través de la ruta física relativa al fichero actual. En otro caso se busca en el directorio `node_modules` y se trata como código de terceros.

>En general no tendrás que preocuparte de estas importaciones físicas, pues el *VSCode* y las extensiones esenciales se encargan de hacerlo automáticamente según lo uses en tu código.

## 1.3 Generación de módulos

Hasta ahora los módulos involucrados son librerías de terceros o que se crearon mágicamente con la aplicación. Es hora de **crear tu primer módulo**. Para eso usaremos otro comando del *cli*, el `ng generate module`. En una ventana del terminal escribe:

```shell
ng g m core
```

Esta es la sintaxis abreviada del comando [`ng generate`](https://github.com/angular/angular-cli/wiki/generate) el cual dispone de varios planos de construcción o *blueprints*. El que he usado aquí es el de `module aka m` para la construcción de módulos.

El resultado es la creación del fichero `core/core.module.ts` con la declaración y decoración del módulo `CoreModule`.
Este módulo te servirá de **contenedor para guardar componentes** y otros servicios esenciales para nuestra aplicación. Pero eso lo veremos más adelante. 

```typescript
@NgModule({
  imports: [],
  declarations: [],
  exports: []
})
export class CoreModule {}
```

Por ahora hay que asegurar que **este módulo es importado por el raíz, el AppModule**. Para ello comprobaremos que la línea de importación del módulo principal esté parecida a esto:

```typescript
@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule, CoreModule],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

> El módulo raíz, al igual que como verás más tarde con el componente ráiz, es un tanto especial. Su nombre oficial es App, aunque todo la documentación se refiere a él como raíz o root. 

# 2. Componentes

Los módulos son contenedores. Lo primero que vamos a guardar en ellos serán componentes. **Los componentes son los bloques básicos de construcción de las páginas web en Angular 6**. Contienen una parte visual en html (la Vista) y una funcional en Typescript (el Controlador).

>La aplicación original que crea el CLI nos regala un primer componente de ejemplo en el fichero `app.component.ts`. Según la configuración del CLI este componente puede haber sido creado en un sólo fichero (es el caso escogido a efectos didácticos) o en dos, tres o incluso cuatro ficheros especializados (con la vista y los estilos en ficheros propios y fichero extra para pruebas unitarias).

## 2.1 Anatomía de un componente

Los componentes, como el resto de artefactos en Angular 6, serán **clases TypeScript decoradas** con funciones específicas. En este caso la función es `@Component()` que recibe un objeto de definición de componente. Igual que en el caso de los módulos contiene las propiedades en las que configurar el componente.

```typescript
import { Core } from "@angular/core";

@Component({
  selector: "app-root",
  template: `<h1>Hello</h1>`,
  styles: []
})
export class AppComponent {}
```

**Los componentes definen nuevas etiquetas HTML** para ser usados dentro de otros componentes. Excepcionalmente en este caso por ser el componente raíz se consume en el página `index.html`. El nombre de la nueva etiqueta se conoce como *selector*. En este caso la propiedad `selector: "app-root"` permite el uso de este componente dentro de otro con esta invocación `<app-root></app-root>`. En este caso el componente raíz.

>Particularidades del compnente raíz. Su nombre oficial es `AppComponent`, y su selector debería llamarse `app-app`. Pero su *selector real* es `app-root`, formado a partir del prefijo de la aplicación y su supuesto nombre oficioso. Observa el prefijo `app` que se usará en todos los componentes propios, fue asignado por defecto durante la generación de la aplicación. Puede personalizarse usando el modificador `--prefix` de `ng new` y en distintos ficheros de configuración. Volviendo al componente raíz; está destinado a ser usado en la página principal, en el `index.html`. Eso obliga a registrarlo de una manera especial en el módulo raíz. Hay que incluirlo en el array `bootstrap: [AppComponent]`, es ahí donde se incluyen los componentes con la capacidad de lanzar *bootstrap* la aplicación. 

```typescript
@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule, CoreModule],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule {}
```
Y en el `index.html`

```html
<body>
  <app-root></app-root>
</body>
```


**La plantilla representa la parte visual** del componente. De forma simplificada, o cuando tiene poco contenido, puede escribirse directamente en la propiedad `template` del objeto decorador. Pero es más frecuente encontrar la plantilla en su propio fichero *html* y referenciarlo como una ruta relativa en la propiedad `templateUrl`.

La propiedad **`styles` y su gemela `stylesUrl` permiten asignar estilos** *CSS, SASS o LESS* al componente. Estos estilos se incrustan durante la compilación en los nodos del *DOM* generado. Son exclusivos del componente y facilitan el diseño y maquetación granular de las aplicaciones.

**En la clase del componente nos encontraremos la implementación de su funcionalidad**. Normalmente expondrá propiedades y métodos para ser consumidos e invocados de forma declarativa desde la vista.

Una aplicación web en Angular 6 se monta como un **árbol de componentes**. El componente raíz ya viene creado y convenientemente declarado; ahora toca darle contenido mediante una estructura de página y las vistas funcionales.

## 2.2 Generación de componentes

Para **crear nuevos componentes** vamos a usar de nuevo el *CLI* con su comando `ng generate component` o abreviadamente `ng g c`. Pero ahora con los planos para construir un componente. La sintaxis completa del [comando *generate*](https://github.com/angular/angular-cli/wiki/generate-component) permite crear componentes en diversas formas.

Casi **todas las páginas tienen una estructura** similar que de forma simplista queda en tres componentes. Uno para la barra de navegación, otro para el pie de página y otro intermedio para el contenido principal. 

Ejecuta en una terminal estos comandos para que generen los componentes y comprueba el resultado en el editor.

```shell
ng g c core/navigator --export
ng g c core/navigator/header --flat
ng g c core/navigator/main --flat
ng g c core/navigator/footer --flat 
```

Fíjate en el componente del fichero `nav.component.ts`. Su estructura es igual a la del componente raíz. Destaca que el nombre del componente coincide con el nombre del selector: `app-navigator` y `NavigatorComponent`. Esto será lo normal a partir de ahora. Sólo el componente raíz tiene la excepción de que su nombre `App` no coincide con su selector `root`.

```typescript
import { Component, OnInit } from '@angular/core';
@Component({
  selector: 'app-navigator',
  template: `
    <app-header></app-header>
    <app-main></app-main>
    <app-footer></app-footer>
  `,
  styles: []
})
export class NavigatorComponent implements OnInit {
  constructor() {}
  ngOnInit() {}
}
```

## 2.3 Componentes públicos y privados

La clave del código limpio es **exponer funcionalidad de manera expresiva pero ocultar la implementación**. Esto es sencillo con los lenguajes de POO, pero en HTML no era nada fácil. Con la **programación basada en componentes** podemos crear pantallas complejas, reutilizables y que a su vez contengan y oculten la complejidad interna a sus consumidores.

Los componentes no deciden por sí mismos su **visibilidad**. Cuando un componente es generado se declara en un módulo contenedor en su propiedad `declares:[]`. Eso lo hace visible y utilizable por cualquier otro componente del mismo módulo. Pero **si quieres usarlo desde fuera tendrás que exportarlo**. Eso se hace en la propiedad `exports:[]` del módulo en el que se crea. 

>La exportación debe hacerse a mano o indicarse con el *flag* `--export` para que lo haga el *cli*. Esto se ha hecho en el componente `navigator` para poder usarlo en el componente `app`.

**Los componentes privados suelen ser sencillos**. A veces son creados para ser específicamente consumidos dentro de otros componentes. En esas situaciones interesa que sean privados y que generen poco ruido. Incluso, en casos extremadamente simples, si usamos el modificador `--flat` ni siquiera generan carpeta propia.

Como regla general, **cuando en una plantilla se incruste otro componente**, Angular lo buscará dentro del propio módulo en el que pretende usarse. Si no lo encuentra entonces lo buscará entre los componentes exportados por los módulos que hayan sido importados por el actual contenedor.

# 3. Organización

Todos los programas tiene partes repetitivas. Los principios de **organización y código limpio** nos permiten identificarlas y reutilizarlas. Con los componentes ocurre lo mismo. El módulo y los componentes recién creados suelen ser comunes a casi todas las aplicaciones. Estos y otros muchos surgirán de manera natural durante el desarrollo de una aplicación para ser utilizados en múltiples páginas. 

Son **componentes de infraestructura**. Conviene guardarlos en una carpeta especial. Aquí la he llamado `shared`, pero `tools`, `common`, o `lib` suelen ser otros nombres habituales.

El caso es **distinguir los componentes de infraestructura de los de negocio** o funcionalidad. Los módulos `core` y `shared` los trataremos como de infraestructura y todos los demás serán de negocio (aún no tenemos). El primero es para meter cosas de uso único esenciales para la aplicación. El segundo para meter bloques reutilizables durante la construcción de la aplicación. Recuerda que sólo son convenios de arquitectura de software; adáptalos a tus necesidades.

![Árbol de módulos](/images/1-base_module_tree.png)

>En esta aplicación hasta ahora no es nada funcional,!y ya tiene tres módulos y cinco componentes!. Puede parecer sobre-ingeniería, pero a la larga le verás sentido. Por ahora te permitirá practicar con la creación de módulos y componentes. Para un ejemplo más realista, consulta cómo está hecho *Astrobot* [AcademiaBinaria/AstroBot](https://github.com/AcademiaBinaria/astrobot/) , que es el hermano mayor y más profesional de *AutoBot*.

Con esto tendrás una base para una aplicación *Angular 6*. Sigue esta serie para añadirle funcionalidad mediante [Páginas y rutas Angular SPA](../paginas-y-rutas-angular-spa/) mientras aprendes a programar con Angular6.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>