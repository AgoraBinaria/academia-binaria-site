---
title: Hola Angular5 CLI
permalink: hola-angular_5-cli
date: 2017-11-07 13:38:42
tags:  
- Angular
- Angular5
- CLI
- Tutorial
categories:
- [Tutorial, Angular5] 
thumbnail: /css/images/angular-5_0_cli_1-5.jpg
---
![Tutorial Angular5 0-CLI1.5](/images/tutorial-angular-5_0_cli_1-5.jpg)

**Angular en su versión 5 es la plataforma perfecta para el desarrollo** profesional de aplicaciones modernas. **El CLI es la herramienta** adecuada para generar aplicaciones Angular. Juntos son imbatibles en cuanto a velocidad en desarrollo y a potencia en ejecución.

<!-- more -->

El comúnmente conocido como **AngularCLI** o *CLI a secas* es la herramienta de línea de comandos estándar para crear, depurar y publicar aplicaciones Angular. En su versión 1.5 es más potente y versátil que nunca. Además es muy sencillo dominar los aspectos básicos.

>Código asociado a este artículo en *GitHub*: [AcademiaBinaria/angular5/0-hello](https://github.com/AcademiaBinaria/angular5/tree/master/0-hello/cash-flow) 

# 1. Instalación de Angular CLI 1.5
Para empezar, como en casi cualquier desarrollo necesitarás disponer de *NodeJS* y su manejador de de paquetes *npm*. Tenerlos actualizados es un mandamiento básico para un desarrollador web.

Empieza con una instalación global que te permita usar la herramienta desde cualquier directorio. Comprueba la versión instalada y accede a la ayuda en línea, tanto general como para el primer comando que vas a usar.

```shell
npm i -g @angular/cli@latest
ng -v
ng help
ng help new
```

# 2. Crear y ejecutar una aplicación Angular 5
Una vez instalado el CLI de manera global, puedes empezar a usarlo en tu directorio de trabajo. El primer comando será `ng new` que te va a **generar toda una aplicación funcional** y las configuraciones necesarias para depuración, pruebas y ejecución.

```shell
ng new cash-flow -p cf --minimal true --routing true 
cd cash-flow
npm start
```
Las líneas anteriores generan una **aplicación mínima pero funcional**. Una vez finalizada la instalación de todas las librerías necesarias puedes bajar a la carpeta recién creada y ejecutar el comando *npm* standard de arranque `npm start`.

Si todo va bien, en unos segundo podrás visitar [http://localhost:4000](http://localhost:4000) para ver en marcha la aplicación.

Pero volvamos a la terminal y analicemos la primera línea. `ng new cash-flow -p cf --minimal true --routing true`. 

> En este tutorial crearemos una aplicación de gestión financiera básica llamada **cash-flow** Una excusa para aprender a programar en Angular, nada serio. El comando generador mostrado utiliza opciones que nos vendrán bien en un futuro, aunque por ahora sólo sirven para demostrar las capacidades del generador. Para empezar podría limitarse a un simple `ng new mi-aplicacion` pero a la alarga vendrá bien usar estas opciones.

| Comando  | Significado |
| -------- | ----------- |
| ng  | programa principal del cli instalado en la máquina  |
| new  | comando para solicitar la generación una nueva aplicación  |
| cash-flow  | nombre de la nueva aplicación  |
| -p  | modificador para establecer un prefijo de nombrado  |
| cf  | valor del prefijo, normalmente las iniciales de la aplicación  |
| --minimal  | la aplicación en su mínima expresión   |
| true  | valor para activar la anterior opción  |
| --routing  |  vamos a creara una aplicación SPA y eso requiere rutas   |
| true  | valor para activar la anterior opción  |

# 3. Estructura de una aplicación Angular
Una vez generada y ejecutada, toca estudiar cómo es la estructura de la aplicación. Para ello revisa carpeta a carpeta. Las malas noticias son que hay **una enorme cantidad de ficheros y carpetas**, las buenas son que como verás, casi todo es **configuración e infraestructura**. 
 

## 3.1 Visual Studio Code
Para ver y editar los ficheros te vale cualquier editor de código, pero yo uso y te recomiendo [VSCode](https://code.visualstudio.com/). Es un gran editor, gratuito y multiplataforma. Viene con un terminal integrado y puedes mejorarlo instalando extensiones.

Para empezar instala un paquete de extensiones ya configurado y preparado para el desarrollo de aplicaciones con *Angular*,  se llama [Angular Essentials](https://marketplace.visualstudio.com/items?itemName=johnpapa.angular-essentials). Con eso y el [Material Icon Theme](https://marketplace.visualstudio.com/items?itemName=PKief.material-icon-theme) verás Angular en colores.

## 3.2 Carpetas y Ficheros principales
Volviendo a la estructura de ficheros y carpetas te encontrás con muchos ficheros de distintos tipos. Por ahora sólo tienes que familiarizarte con estos:

* .angular-cli.  *: configuración del CLI*
* package.json *: dependencias de librerías y scripts*
* src/ *: la carpeta donde están los archivos fuentes*
    * index.html *: el fichero HTML índice*
    * main.ts *: fichero TypeScript de arranque de la aplicación*
    * app/ *: la carpeta con le código específico de tu aplicación*
        * app.module.ts *: las aplicaciones son árboles de módulos, y este es su raíz*
        * app.component.ts *: las páginas son árboles de componentes, y este es su raíz*

Echa un vistazo a estos ficheros, pronto los modificaremos para sentirnos programadores.

# 4. Edición  
Angular CLI instala y configura un montón de herramientas que te harán la vida más fácil. Entre otras la capacidad de recargar la aplicación en caliente, según guardes tu trabajo como programador. En esta última versión 1.5 ha mejorado y es realmente rápido.

Para probarlo sólo tienes que dejar arrancada la aplicación con el comando `npm start` cambiar un fichero y comprobar el resultado. Te propongo empezar como en cualquier otro lenguaje, por el famoso *hola mundo*.

## 4.1 Hola Mundo
Abre el fichero `app.component.ts` y busca dentro de la clase `AppComponent` la propiedad `title`. Asígnale el saludo de rigor `title = 'Hello World';`, guarda y comprueba cómo tu navegador se habrá actualizado automáticamente.

Toda esta magia depende de una cadena de comandos que lanzan herramientas previamente instaladas y configuradas por el CLI.

1. npm start
2. ng serve
    1. webpack server en http://localhost:4000
    2. vigilancia de cambios obre la carpeta src/
    3. livereload
      1. recompilado de la aplicación
      2. recarga del navegador

Esto es sólo el principio, Angular CLI puede hacer mucho más por ti. Descúbrelo en su [wiki](https://github.com/angular/angular-cli/wiki) o sigue esta serie para usarlo y aprender a programar con Angular.

> Aprender, programar, disfrutar, repetir.
