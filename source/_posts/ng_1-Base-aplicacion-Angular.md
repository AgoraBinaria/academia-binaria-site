---
title: Base para una aplicación Angular
permalink: base-aplicacion-angular
date: 2017-11-09 11:09:46
tags:  
- Angular
- Angular5
- CLI
- Tutorial
categories:
- [Tutorial, Angular] 
thumbnail: /css/images/angular-5_1_Base.jpg
---
![Tutorial Angular5 1-Base](/images/tutorial-angular-5_1_base.jpg)

Vamos a crear una **base sobre la que programar una aplicación Angular 5** profesional. Usaremos el *CLI* para generar una estructura sobre la que crecer. Será como una semilla para un desarrollo controlado. 
La idea de árbol se usa en muchas analogías informáticas. La emplearemos en dos conceptos básicos en Angular: **los módulos y los componentes**.

<!-- more -->

Partimos de la aplicación tal cómo la dejamos en el [Hola Mundo en Angular](../hola-angular-cli/). Al finalizar tendrás un esqueleto del que colgar módulos y componentes funcionales.

>Código asociado a este artículo en *GitHub*: [AcademiaBinaria/angular5/1-base](https://github.com/AcademiaBinaria/angular5/tree/master/1-base/cash-flow) 


# 1. Módulos

Los módulos son **contenedores dónde almacenar los componentes y servicios** de una aplicación. En Angular cada programa se puede ver como un árbol de módulos jerárquico. A partir de un módulo raíz se enlazan otros módulos en un proceso llamado importación.


## 1.1 Definición mediante decoradores

Antes de importar cualquier módulo hay que definirlo. En Angular **los módulos de declaran como clases de TypeScript**, habitualmente vacías, decoradas con una función especial. Es la función `@NgModule()` que recibe un objeto como único argumento. En las propiedades de ese objeto es dónde se configura el módulo. 

Mira el módulo `AppModule` original que genera el CLI en el fichero `app.module.ts`.

```typescript
@NgModule({
  declarations: [AppComponent],
  imports: [AppRoutingModule, BrowserModule],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

## 1.2 Importación de otros módulos

El módulo `App` también se conoce como **módulo raíz** porque de él surgen las demás ramas que conforman una aplicación. La asignación de los nodos hijos se realiza en la propiedad `imports:[]`, que es un array de punteros a otros módulos.

>En la situación original el módulo principal depende un módulo para realizar el enrutado (el `AppRoutingModule` que usarás más adelante) y de otro para la presentación en el navegador (el `BrowserModule`).


### 1.2.1 Dos mundos paralelos: imports de Angular e import de TypeScript

Si es la primera vez que ves código TypeScript te llamarán la atención las primeras líneas de cada fichero. En el `app.module.ts` son algo así:

```typescript
import { BrowserModule } from "@angular/platform-browser";
import { NgModule } from "@angular/core";
import { AppRoutingModule } from "./app-routing.module";
import { AppComponent } from "./app.component";
```

Estas **sentencias de importación son propias del lenguaje** y nada tienen que ver con Angular. En ellas se indica que este fichero importa el contenido de otros ficheros *TypeScript*. La importación se realiza a través de la ruta física relativa al fichero actual, o en el directorio `node_modules` si es código de terceros.

>En general no tendrás que preocuparte de estas importaciones físicas, pues el *VSCode* y las extensiones esenciales se encargan de hacerlo automáticamente.  


## 1.3 Generación de módulos

Hasta ahora los módulos involucrados son librerías de terceros o se crearon mágicamente con la aplicación. Es hora de **crear tu primer módulo**. Para eso usaremos otro comando del *cli*, el `ng generate module`. En una ventana del terminal escribe:

```shell
ng g m lib/components
```

Esta es la sintaxis abreviada del comando [`ng generate`](https://github.com/angular/angular-cli/wiki/generate) el cual dispone de varios planos de construcción o *blueprints*. El que he usado aquí es el de `module` para la construcción de módulos.

El resultado es la creación del fichero `lib/components/components.module.ts` con la declaración y decoración del módulo `ComponentsModule`.
Este módulo te servirá de **contenedor para guardar componentes** como veremos más adelante. 

```typescript
@NgModule({
  imports: [],
  declarations: [],
  exports: []
})
export class ComponentsModule {}
```

Por ahora hay asegurar que **este módulo es importado por el raíz**. Para ello comprobaremos que la línea de importación del módulo principal esté parecida a esto:

```typescript
@NgModule({
 imports: [AppRoutingModule, BrowserModule, ComponentsModule],
})
```

# 2. Componentes

Los módulos son contenedores. Lo primero que vamos a guardar en ellos serán componentes. **Los componentes son los bloques básicos de construcción de las páginas web en Angular**. Contienen una parte visual en html (la Vista) y una funcional en Typescript (el Controlador).

>La aplicación original que crea el CLI nos regala un primer componente de ejemplo en el fichero `app.component.ts`. Según la configuración del CLI este componente puede haber sido creado en un sólo fichero (es el caso escogido a efectos didácticos) o en dos o tres ficheros especializados (con la vista y los estilos en ficheros propios).


## 2.1 Anatomía de un componente

Los componentes, como el resto de artefactos en Angular, serán **clases TypeScript decoradas** con funciones específicas. En este caso la función es `@Component()` que recibe un objeto de definición de componente. Igual que en el caso de los módulos contiene las propiedades en las que configurar el componente.

```typescript
import { Component } from "@angular/core";

@Component({
  selector: "cf-root",
  template: ``,
  styles: []
})
export class AppComponent {}
```

**Los componentes definen nuevas etiquetas HTML** para ser usados dentro de otros componentes. Excepcionalmente en este caso por ser el componente raíz se consume en el página `index.html`. El nombre de la nueva etiqueta se conoce como *selector*. En este caso la propiedad `selector: "cf-root"` permite el uso de este componente dentro de otro con esta invocación `<cf-root></cf-root>`. 

>Observa el prefijo `cf` que en este se corresponde con *cash-flow* el nombre de la aplicación. El prefijo fue asignado durante la generación de la aplicación y se usará en todos los componentes propios para evitar colisiones con terceros.

**La plantilla representa la parte visual** del componente. De forma simplificada, o cuando tiene poco contenido, puede escribirse directamente en la propiedad `template` del objeto decorador. Pero es más frecuente encontrar la plantilla en su propio fichero *html* y referenciarlo como una ruta relativa en la propiedad `templateUrl`.

La propiedad **`styles` y su gemela `stylesUrl` permiten asignar estilos** *CSS, SASS o LESS* al componente. Estos estilos se incrustan durante la compilación en los nodos del *DOM* generado. Son exclusivos del componente y facilitan el desarrollo granular de aplicaciones.

**En la clase del componente nos encontraremos la implementación de su funcionalidad**. Normalmente expondrá propiedades y métodos para ser consumidos e invocados de forma declarativa desde la vista.

Una aplicación web en Angular se monta como un **árbol de componentes**. El componente raíz ya viene creado; ahora toca darle contenido mediante una estructura de página y las vistas funcionales.


## 2.2 Generación de componentes

Para **crear nuevos componentes** vamos a usar de nuevo el *CLI* con su comando `ng generate component`. Pero ahora con los planos para construir un componente. La sintaxis completa del [comando *generate*](https://github.com/angular/angular-cli/wiki/generate-component) permite crear componentes en diversas formas.

Casi **todas las páginas tienen una estructura** similar que de forma simplista queda en tres componentes. Uno para la barra de navegación, otro para el pie de página y otro intermedio para el contenido principal. 

Ejecuta en una terminal estos comandos para que generen los componentes y comprueba el resultado en el editor.

```shell
ng g c lib/components/nav 
ng g c lib/components/footer --export 
```

Fíjate en el componente del fichero `nav.component.ts`. Su estructura es igual a la del componente raíz. Destaca que el nombre del componente coincide con el nombre del selector: `cf-nav` y `NavComponent`. Esto será lo normal a partir de ahora. Sólo el componente raíz tiene la excepción de que su nombre `App` no coincide con us selector `root`.

```typescript
@Component({
  selector: "cf-nav",
  template: ``,
  styles: []
})
export class NavComponent {}
```

## 2.3 Componentes públicos y privados

La clave del código limpio es **exponer funcionalidad de manera expresiva pero ocultar la implementación**. Esto es sencillo con los lenguajes de POO, pero en HTML no era nada fácil. Con la **programación basada en componentes** podemos crear pantallas complejas, reutilizables, que a su vez contengan y oculten la complejidad interna a sus consumidores.

Los componentes no deciden por sí mismos su **visibilidad**. Cuando un componente es generado se declara en un módulo contenedor en su propiedad `declares:[]`. Eso lo hace visible y utilizable por cualquier otro componente del mismo módulo. Pero **si quieres usarlo desde fuera tendrás que exportarlo**. Eso se hace en la propiedad `exports:[]` del módulo en el que se crea. 

>La exportación debe hacerse a mano o indicarse con el *flag* `--export` para que lo haga el *cli*. Esto se ha hecho en el componente `footer`.

**Los componentes privados suelen ser sencillos**. A veces son creados para ser específicamente consumidos dentro de otros componentes. en esas situaciones interesa que sean privados y que generen poco ruido. 

```shell
ng g c lib/components/nav/title --flat
```
>El anterior comando crea uno de esos componentes, que es visible dentro del módulo que lo declara, pero no lo és fuera de él (no lleva `--export`). Usando el atributo `--flat` además le pides no cree una carpeta específica.

# 3. Organización

Todos los programas tiene partes repetitivas. Los principios de **organización y código limpio** nos permiten identificarlas y reutilizarlas. Con los componentes ocurre lo mismo. El módulo y los componentes recién creados suelen ser comunes a casi todas las aplicaciones. Estos y otros muchos surgirán de manera natural durante el desarrollo de una aplicación para ser utilizados en múltiples páginas. 

Son **componentes de infraestructura**. Conviene guardarlos en una carpeta especial. Aquí la he llamado `lib`, pero `tools`, `common`, `shared` suelen ser otros nombres habituales.

El caso es **distinguir los componentes de infraestructura de los de negocio** o funcionalidad. Estos últimos yo los agrupé en una carpeta llamada `views`. De nuevo escoge un nombre que te resulte significativo. Hay otras alternativas muy usadas como `pages`, `routes`...

>En esta aplicación hasta ahora nada funcional, te propongo **crear un módulo funcional** con un único componente para mostrar el contenido de la página *home* de la aplicación. Puede parecer sobre-ingeniería, pero a la larga le verás sentido, y por ahora te permitirá practicar con la creación de módulos y componentes.

```shell
ng g m views/home
ng g c views/home/home --export --flat
```

El fichero `home.module.ts` que define el módulo debe quedar asi:

```typescript
@NgModule({
  imports: [
    CommonModule
  ],
  declarations: [HomeComponent],
  exports: [HomeComponent]
})
export class HomeModule { }
```
En el componente podemos empezar a usar las capacidades de Angular para crear páginas web de contenido dinámico.

```typescript
import { Component, OnInit } from "@angular/core";
import * as moment from "moment";
@Component({
  selector: "cf-home",
  template: `
    <main>
      <header>Main content of the Home Page</header>
      <div>Still a work in progress... for Now: {{ now }}</div>
      <div>... to be continued... Tomorrow: {{ tomorrow }}</div>
    </main>
  `,
  styles: []
})
export class HomeComponent implements OnInit {
  now = moment().format();
  tomorrow = moment().add(1, "days").format();
  constructor() {}

  ngOnInit() {}
}
```

Ahora que ya tenemos **componentes de infraestructura y negocio**, podremos usarlos como bloques constructores de alto nivel en el componente raíz.

```typescript
@Component({
  selector: "cf-root",
  template: `
    <cf-nav></cf-nav>
    <cf-home></cf-home>
    <cf-footer></cf-footer>  
  `,
  styles: []
})
export class MainComponent {}
```


Con esto tendrás una base para una aplicación *Angular*. Sigue esta serie para añadirle funcionalidad mientras aprendes a programar con Angular5.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>