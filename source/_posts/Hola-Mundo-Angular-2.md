---
title: Hola Mundo Angular 2
tags:  
- Angular2
categories:
- Introducción 
permalink: hola-mundo-en-angular-2
id: 12
updated: '2016-10-31 12:36:28'
date: 2016-05-18 16:10:18
---

>Código asociado en GitHub: [angular2/1-HolaMundo/](https://github.com/AcademiaBinaria/angular2/tree/master/1-hola-mundo)

Qué lejanos aquellos tiempos dónde un [Hola Mundo en AngularJS](https://github.com/AcademiaBinaria/HolaAngularJS) se hacía en 2 líneas de código. Ahora necesitaré miles de ficheros y media hora de explicación. Pero el resultado valdrá la pena. Estaremos en la pista de lanzamiento para crear **aplicaciones de nivel empresarial con Angular 2.**

Ya he explicado que Angular ha pasado [de framework a plataforma](http://academia-binaria.com/angular2-primeras-impresiones/), y que ya no es para aficionados. Grandes desarrollos en equipo requieren **herramientas y procedimientos** a la altura. La primera opción que te recomiendo es [Angular CLI](https://cli.angular.io/), un generador de aplicaciones trufado de buenas prácticas y procedimientos. Por raro que te parezca, cualquier otra opción es *aún* más compleja que la que te muestro.


## Preparando el entorno
Las herramientas que voy a usar requieren [NodeJS](nodejs.org). Te recomiendo que instales una de sus últimas versiones. Tras la instalación tendrás acceso a [npm](npmjs.com) para poder instalar librerías y utilidades como Angular CLI.

```
npm install -g angular-cli
```

A partir de ahora en tu linea de comandos podrás usar el programa `ng` seguido de algún comando como `new generate serve lint test e2e build`. Usaremos algunos en esta demo.

## Creando aplicaciones y componentes
Escoge un directorio en un disco con espacio libre. No es broma, hasta 300 mb o más para empezar. Tranquilo, en distribución la cosa pinta mucho mejor y Angular es muy ligero. Ahora teclea:


```
ng new hola-angular-2
```

Unos segundos o minutos más tarde... podrás abrir la recién creada carpeta, yo lo hago con [VSCode](https://code.visualstudio.com/), y explorar el contenido de sus 3 directorios y miles de ficheros!!!

Después del susto, tranquilidad de nuevo. La mayoría son dependencias de terceros productos, herramientas necesarias para ejecutar aplicación o para alguno de sus procesos de *test, lint o distribución*. Centrémonos en el directorio `src` dónde están los fuentes, el código que tendríamos que haber creado nosotros.

De un primer vistazo puede que sólo reconozcas al viejo `index.html`. No te agobies, por ahora es suficiente. Contendrá algo como esto:

```html
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <title>HolaAngular2</title>
  <base href="/">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="icon" type="image/x-icon" href="favicon.ico">
</head>
<body>
  <!--Componente raíz de la aplicación-->
  <app-root>Loading...</app-root>
</body>
</html>
```

De lo que es Angular 2 lo único interesante es el componente `<app-root>Loading...</app-root>`. Todo, incluida la aplicación principal, debe ser definido y declarado como un componente. De hecho definiremos las aplicaciones Angular2 como árboles de componentes. Y tdodo árbol debe tener una raíz. Mientras Angular no entre en fucionamiento, el susario verá el mensaje de *Loading...* después la magia de Angular2 lo sustituirá por el contenido del componente `app-root` predefinido por el generador. 

## TypeScript
Sin entrar en debates de [qué lenguaje usar para programar en Angular2](https://www.youtube.com/watch?v=OpS2R7rbpRg) te resumo mi posición: 

1.- TypeScript te permite anotar tu **JavaScript con tipos**. Esto tiene dos ventajas: *intellisense* mientras codificas y chequeo de tipos cuando compilas.
2.- TypeScript es la **única opción automatizada a día de hoy** con Angular CLI.

Por lo demás no hay porqué alarmarse. [TypeScript](https://www.typescriptlang.org/) es un *superset de JavaScript ES6* con unas mejoras evidentes que no tardarás en dominar. Eso si, tienes que aprender [JavaScript ES2015 o ES6](http://es6-features.org/).

Sabiendo esto, entra sin miedo en cualquier fichero de extensión `.ts` y verás que es muy parecido a cualquier `.js` de la nueva versión. Por ejemplo en el citado `main.ts` aparecerá algo así:

```javascript
// importaciones de dependencias TypeScript al estilo ES6
// primero los básicos para compatibilidad con navegadores
import './polyfills.ts';
// luego cosas de Angular
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';
import { enableProdMode } from '@angular/core';
// después nuestro código, empezando por la configuración
import { environment } from './environments/environment';
// y por último la aplicación a leída desde el módulo raíz, llamado app por convenio.
import { AppModule } from './app/';

// condiciones para ejecutar en modo desarrollo o producción
if (environment.production) {
  enableProdMode();
}
// arranque de la aplicación invocando al módulo raíz
platformBrowserDynamic().bootstrapModule(AppModule);
```

Centrándonos en el código que habremos de mantener fíjate en la línea `import { AppModule } from './app/';`. Le indica a *WebPack* que importe el contenido de la carpeta `./app/`. Para ello buscará en dicho directorio un archivo `index.ts`. Ese fichero sirve de índice y contiene las instrucciones para exportar el código interesante del resto de la carpeta. En nuestro caso son el módulo y el componente raíz. 

## El módulo raíz
Las aplicaciones Angular2 están pensadas para crecer. Para ello es fundamental cierto grado de modularidad. EL viejo `angular.module` ha vuelto en la versión 2. Mira dentro del fichero `app.module.ts` y verás código similar a este:

```javascript
// objetos con utilidades comunes del framework
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { HttpModule } from '@angular/http';
// importación del componente raíz, definido en esta misma carpeta
import { AppComponent } from './app.component';
// decorador con metadata que define un módulo
@NgModule({
  declarations: [
    AppComponent
  ], // cosas declaradas en este módulo
  imports: [
    BrowserModule,
    FormsModule,
    HttpModule
  ], // otros módulos que necesitamos para que este funcione
  providers: [] , // inyección de servicios comunes para la aplicación
  bootstrap: [AppComponent] // componente raíz para el arranque
})
// los módulos son clases contendoras 
// habitualmente con poco o ningún código
export class AppModule { }
```

Un módulo no es más que una clase contenedora. Cada módulo puede incluir múltiples componentes y servicios. Normalmente un módulo dependerá de otros. El módulo raíz declara un componente especial para el arranque de la aplicación: *El componente raíz*

## El componente raíz
Buceando a mayor profundidad nos encontramos con el resto del contenido de la carpeta `./app/`. Son archivos con nombres tipo `app.component.*` y se usan para definir un componente.

Los **componentes son los bloques de construcción de Angular 2** que representan regiones de la pantalla. Las aplicaciones se definen como árboles de componentes. Nuestra aplicación es un árbol que tiene una raíz, habitualmente llamado `app` y que es común a cualquier desarrollo.

Cada componente a su vez está formado por tres partes:
1. **La vista**: es el código que se renderizará para los usuarios. Esta plantilla estará en un fichero de extensión `.html`.
2. **La clase controladora**: En ES6 usaremos clases para declarar los controladores que exponen datos y funcionalidad a la vista.
3. **Metadata**: Se declara como un decorador, una función especial de TypeScript, que recibe un objeto de configuración. Esto acompaña al controlador en un fichero de extensión `.ts`

Empecemos por este último fichero, el `app.component.ts`.
```javascript
import { Component } from '@angular/core';
// Función decoradora que registra un componente
@Component({
  selector: 'app-root', // elemento html consumidor
  templateUrl: './app.component.html', // ruta relativa a la vista
  styleUrls: ['./app.component.css'] // potencialmente múltiples hojas de estilo
})
// clase que representa un controlador 
// con su modelo de datos (title ) y métodos de acción (aún no tiene)
// Esta clase es todo lo que se exporta en este fichero
// y esto se importará en app.module.ts para ser incorporado el módulo raíz
export class AppComponent {
  // las propiedades de la clase representan el modelo de datos
  // son accesibles desde la vista
  title = 'app works!';
}
```

Seguro que la parte más novedosa es `@Component({...})`. Es el equivalente a los antiguos Objetos de Definición de Directivas. Lo que hace es asociar al controlador una plantilla HTML `app.component.html` y un selector para ser invocado desde otra vista `<app-root></app-root>`. El resto por ahora puedes obviarlo.

Y hablando de la plantilla, echemos un vistazo a `app.component.html`. Contendrá algo así:

```html
<h1>
  <!--Interpolación de variables definidas en el modelo del componente-->
  {{ title }}
</h1>
```

Estas son cosas que te resultarán muy familiares como la interpolación `{{ title }}` que permite mostrar el famoso *app works!*, nueva versión del *hola mundo*. Ya está, el resto ya es sólo usar este componente en el `index.html`, 
Recuerda:
```html
  <!--Componente raíz de la aplicación-->
  <app-root>Loading...</app-root>
```
## Angular2 en acción
Para lanzar y probar tu aplicación necesitas otro comando de Angular-CLI. Este comando se ocupa entre otras cosas de todo el proceso necesario para **transformar el código TypeScript en JavaScript** reconocible por el navegador. También crea un mini servidor estático y además refresca el navegador a cada cambio los fuentes. Un salvavidas para un recién llegado a Angular 2. Teclea en tu terminal:

```
ng serve
```

Si todo ha ido bien, no siempre ocurre con estas versiones tan verdes, podrás disfrutar de **tu primera aplicación con Angular 2** en [http://localhost:4200](http://localhost:4200)  


>[Este vídeo](https://youtu.be/Y7izsxhPpQY) emitido con la colaboración de DesarrolloWeb.com contiene una explicación mas extensa del proceso de trabajo con Angular 2 CLI.



<div class="flex-video">  
    <iframe src="https://www.youtube.com/embed/Y7izsxhPpQY"></iframe>
</div>   