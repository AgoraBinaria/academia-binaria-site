---
title: Hola Angular CLI
permalink: hola-angular-cli
date: 2017-11-07 13:38:42
updated: 2017-11-08 09:47:00
tags:  
- Angular
- Angular5
- CLI
- Tutorial
categories:
- [Tutorial, Angular] 
thumbnail: /css/images/angular-5_0_cli_1-5.jpg
---
![Tutorial Angular5 0-CLI1.5](/images/tutorial-angular-5_0_cli.png)

**Angular en su versión 5 es la plataforma perfecta para el desarrollo** profesional de aplicaciones modernas. **El CLI es la herramienta** adecuada para generar aplicaciones Angular. Juntos son imbatibles en cuanto a velocidad en desarrollo y a potencia en ejecución.

<!-- more -->

El comúnmente conocido como **AngularCLI** o *CLI a secas* es la herramienta de línea de comandos estándar para **crear, depurar y publicar aplicaciones Angular**. En su versión 1.5 es más potente y versátil que nunca. Además es muy sencillo dominar los aspectos básicos.

>Código asociado a este artículo en *GitHub*: [AcademiaBinaria/angular5/0-hello](https://github.com/AcademiaBinaria/angular5/tree/master/0-hello/cash-flow) 

# 1. Instalación de Angular CLI 1.5
Para empezar, como en casi cualquier desarrollo necesitarás disponer de *NodeJS* y su manejador de de paquetes *npm*. Tenerlos actualizados es un mandamiento básico para un desarrollador web.

Empieza con una instalación global que te permita usar la herramienta desde cualquier directorio. Comprueba la versión instalada y accede a la ayuda en línea. La ayuda está disponible tanto de modo general como para cada comando que vayas a usar.

```shell
npm i -g @angular/cli@latest
ng -v
ng help
ng help new
```

# 2. Crear y ejecutar una aplicación Angular 5
Una vez que hayas instalado el CLI de manera global ya puedes empezar a usarlo en tu directorio de trabajo. El primer comando será `ng new` que te va a **generar toda una aplicación funcional** y las configuraciones necesarias para su depuración, pruebas y ejecución.

```shell
ng new cash-flow -p cf --minimal true --routing true 
cd cash-flow
npm start
```
 Una vez finalizada la instalación de todas las librerías necesarias puedes bajar a la carpeta recién creada y ejecutar el comando standard de*npm*  para el arranque de cualquier aplicación: `npm start`.

Si todo va bien, en unos segundo podrás visitar [http://localhost:4000](http://localhost:4000) para ver en marcha la aplicación.

Pero volvamos a la terminal y analicemos la primera línea. `ng new cash-flow -p cf --minimal true --routing true`. 

> En este tutorial crearemos una aplicación de gestión financiera básica llamada **cash-flow** Una excusa para aprender a programar en Angular; nada serio. El comando [`ng new`](https://github.com/angular/angular-cli/wiki/new) mostrado utiliza opciones que nos vendrán bien en un futuro, aunque por ahora sólo sirven para demostrar las capacidades del generador. Para empezar podríamos habernos limitado a un simple `ng new nombre-de-mi-aplicacion` pero a la larga vendrá bien usar estas opciones para la práctica que acompaña a este tutorial.

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
Una vez generada y comprobada la ejecución, toca estudiar cómo es la estructura de la aplicación. Para ello revisa carpeta a carpeta. Las malas noticias son que hay **una enorme cantidad de ficheros y carpetas**, las buenas son que como verás, casi todo es **configuración e infraestructura**. 
 

## 3.1 Visual Studio Code
Para ver y editar los ficheros te vale cualquier editor de código, pero yo uso y te recomiendo [VSCode](https://code.visualstudio.com/). Es un **gran editor, gratuito y multiplataforma**. Viene con un terminal integrado y puedes mejorarlo instalando extensiones.

Antes de empezar debes instalar un paquete de extensiones ya configurado y preparado para el desarrollo de aplicaciones con *Angular*,  se llama [Angular Essentials](https://marketplace.visualstudio.com/items?itemName=johnpapa.angular-essentials). Con eso y el [Material Icon Theme](https://marketplace.visualstudio.com/items?itemName=PKief.material-icon-theme) verás *Angular en colores*.

## 3.2 Carpetas y Ficheros principales
Volviendo a la **estructura de ficheros y carpetas** te encontrás con muchos archivos de distintos tipos. Si eres completamente nuevo en Angular, te llamará la atención las extensiones `.ts`. Son para ficheros [*TypeScript*](https://www.typescriptlang.org/), una evolución del *JavaScript* con facilidades para el programador. Por ahora sólo tienes que familiarizarte con estos:

+ .angular-cli.json  *: configuración del propio CLI*
+ package.json *: dependencias de librerías y scripts*
+ src/ *: la carpeta donde están los archivos fuentes*
    + index.html *: un fichero HTML índice estándar*
    + main.ts *: fichero TypeScript de arranque de la aplicación*
    + app/ *: la carpeta con el código específico de tu aplicación*
        + app.module.ts *: las aplicaciones son árboles de módulos, y este es su raíz*
        + app.component.ts *: las páginas son árboles de componentes, y este es su raíz*

Echa un vistazo a estos ficheros, pronto los modificaremos para sentirnos programadores.

# 4. Edición  
Angular CLI instala y configura un conjunto de herramientas que te harán la vida más fácil. Entre otras, destaca la capacidad de **recargar la aplicación en caliente** en cuanta guardas tu trabajo como programador. En esta última versión, la 1.5, se ha mejorado el proceso y es realmente rápido.

Para probarlo sólo tienes que dejar arrancada la aplicación con el comando `npm start`; **cambiar un fichero de código y comprobar el resultado** en el navegador. Te propongo empezar como en cualquier otro lenguaje; por el famoso *hola mundo*.

## 4.1 Hola Mundo
Abre el fichero `app.component.ts` y busca dentro de él una clase llamada `AppComponent`. Encontrarás que tiene una propiedad `title`. Asígnale el saludo de rigor: `title = 'Hello World';`. Guarda y comprueba cómo tu navegador **se habrá actualizado automáticamente**.

Toda **esta magia depende de una cadena de comandos** que lanzan distintas herramientas previamente instaladas y configuradas por el CLI. Entre ellas cabe mencionar a [*WebPack*](https://webpack.github.io/), un coloso que afortunadamente viene instalado y preparado para funcionar de manera transparente.

Esta es una lista no exhaustiva de lo que sucede.

1. npm start
2. ng serve
3. webpack server en http://localhost:4000
    1. vigilancia de cambios sobre la carpeta src/
    2. livereload
      1. compilado de la aplicación
      2. recarga del navegador


# 5. Configuración

El CLI viene con pilas incluidas, se puede usar desde el primer momento. Sólo quedan pequeñas mejoras que hacer. Por ejemplo ajustar el `package.json` y agregar librerías de terceros.


## 5.1 Package.json

El `package.json` es el fichero estándar de *npm* donde se almacenan las **dependencias de terceros**. Contiene las librerías que necesita la aplicación para ejecutarse, por ejemplo todas las de *Angular*. Y también las herramientas que necesita el programador, por ejemplo el propio *AngularCLI*;
```json
{
  "dependencies": {
      "@angular/core": "^5.0.0",
  },
  "devDependencies": {
      "@angular/cli": "1.5.0",
  }
}
```

Otro uso del `package.json` es servir de **contenedor de scripts** para automatizar tareas de operaciones rutinarias. Por ejemplo, el comando estándar `npm start` ejecutará el contenido asignado en el fichero *json*, originalmente `ng serve`. Esto lanza el servidor de pruebas con sus opciones por defecto. 

Pero el **comando [ng serve](https://github.com/angular/angular-cli/wiki/serve)** admite muchas configuraciones. Te propongo que uses esta para activar un modo de compilación más rápido y seguro, y para que se abra el navegador de forma automática en cuanto lances el servidor.
```json
{
 "start": "ng serve --aot -o",
}
```

## 5.2 Estilos y librerías de terceros
Las librerías que vienen de fábrica tienen todo lo necesario para crear aplicaciones. Pero raro es el caso en que no necesitemos **algún que otro producto de terceros**. Ya sean utilidades como *[Moment](https://momentjs.com/)*, librerías gráficas como *[chart.js](http://www.chartjs.org/)* o la aplicación de estilos y componentes visuales de *frameworks como Bootstrap o MaterialDesign*. Pero todos se instalan de igual forma. Descargándolos con *npm* y adjuntándolos en el `.angular-cli.json`. 

>En este tutorial te propongo usar una hoja de estilos muy simple que mejora la apariencia de cualquier aplicación sin necesidad de usar clases propias. Se llama *[milligram](https://milligram.io/)* y es apropiada para prototipos, pruebas o pequeños proyectos. También usaremos *[Moment](https://momentjs.com/)* para el trabajo con fechas y horas.

Se descargan e instalan de manera estándar.
```shell
npm i milligram --save
npm install moment --save
```

Para que se incluyan en la distribución hay que ir a la configuración del *CLI*.Y se agrega a través del fichero `.angular-cli.json` a la colección de *styles* o de *scripts* que corresponda.
```json
{
  "styles": [
      "../node_modules/milligram/dist/milligram.min.css",
      "styles.css"
      ],
  "scripts": ["../node_modules/moment/min/moment.min.js"]
}
```
Estas colecciones de archivos los usa el *cli* a través de *webpack* para incluirlos **minificados y concatenados en un fichero *bundle* sustituyendo a las clásicas etiquetas html**. De esta forma el fichero `index.html` apenas tendrás que tocarlo. Todo, el html y sus estilos, se construirá en el cliente a partir de instrucciones JavaScript. 

Una cosa más, los cambios en los ficheros de configuración no se auto recargan. Tienes que parar la servidor y volver a lanzarlo para apreciar el estilo *milligram*. 



Esto es sólo el principio, *Angular CLI* puede hacer mucho más por ti. Descúbrelo en su [wiki](https://github.com/angular/angular-cli/wiki) o sigue esta serie, [Base para una aplicación Angular](../base-aplicacion-angular/), para usarlo mientras aprendes a programar con Angular5.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>