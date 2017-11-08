---
title: Base para una aplicación Angular 5
date: 2017-11-08 17:00:09
tags:  
- Angular
- Angular5
- CLI
- Tutorial
categories:
- [Tutorial, Angular5] 
thumbnail: /css/images/angular-5_1_Base.jpg
---
![Tutorial Angular5 0-CLI1.5](/images/tutorial-angular-5_1_base.jpg)

Vamos a crear una **base sobre la que programar una aplicación Angular 5** profesional. Usaremos el *CLI* y *npm* para configurarla y darle una estructura sobre la que crecer. 
La idea de árbol se usa en muchas analogías informáticas. La emplearemos en dos conceptos básicos en Angular: **los módulos y los componentes**.

<!-- more -->

Partimos de la aplicación tal cómo la dejamos en el [Hola Mundo en Angular](). Al finalizar tendrás un esqueleto del que colgara módulos y componentes funcionales.

>Código asociado a este artículo en *GitHub*: [AcademiaBinaria/angular5/1-base](https://github.com/AcademiaBinaria/angular5/tree/master/1-base/cash-flow) 

# 1. Configuración
El CLI viene con pilas incluidas, se puede usar desde el primer momento. Sólo quedan pequeñas mejoras que hacer. Por ejemplo ajustar el packcge json y agregar librerías de terceros.

## 1.1 Package.json

El `package.json` es el fichero estándar de *npm* dónde se almacenan las **dependencias de terceros**. Contiene las librerías que necesita la aplicación para ejecutarse, por ejemplo todas las de *Angular*. Y también las herramientas que necesita el programador, por ejemplo *AngularCLI*;
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

Otro uso del `package.json` es servir de **contenedor de scripts** para automatizar tareas de operaciones rutinarias. Pore ejemplo, el comando estándar `npm start` ejecutará el contenido asignado en el fichero *json*, originalmente `ng serve`. Esto lanza el servidor de pruebas con sus opciones por defecto. 

Pero el **comando [ng serve](https://github.com/angular/angular-cli/wiki/serve)** admite muchas configuraciones. Te propongo que uses esta para activar un modo de compilación más rápido y seguro, y para que se abra el navegador de forma automática en cuanto lances el servidor.
```json
{
 "start": "ng serve --aot -o",
}
```

## 1.2 Estilos y librerías de terceros
Las librerías que viene de fábrica tienen todo lo necesario para crear aplicaciones. Pero raro es el caso que no necesitemos algún otro producto de terceros. Ya sean utilidades como *[Moment](https://momentjs.com/)*, librerías gráficas como *[chart.js](http://www.chartjs.org/)* o la gestión de estilos y componentes visuales de *frameworks como Bootstrap o MaterialDesign*. Pero todos se instalan de igual forma. Descargándolos con *npm* y adjuntándolos en el `.angular-cli.json`. 

Te propongo usar una hoja de estilos muy simple que mejora la apariencia de cualquier aplicación sin usar clases propias. Se llama *[milligram](https://milligram.io/)* y es apropiada para prototipos, pruebas o pequeños proyectos.

Se descarga de manera estándar.
```shell
npm i milligram --save
```

Y se agrega a través del fichero `.angular-cli.json` a la colección de *styles* o de *scripts* si los tuviera.
```json
{
  "styles": [
      "../node_modules/milligram/dist/milligram.min.css",
      "styles.css"
      ],
  "scripts": [],
}
```
Estas colecciones de archivos los usa el *cli* a través de *webpack* para incluirlos minificados y concatenados en un fichero *bundle* sustituyendo a las clásicas etiquetas html. De esta forma el fichero `index.html` apenas tendrás que tocarlo.  

Una cosa más, los cambios en los ficheros de configuración no se auto recargan. Tienes que parar la servidor y volver a lanzarlo para apreciar el estilo *milligram*. 


# 2. Módulos
Los módulos son contenedores dónde almacenar los componentes y servicios de una aplicación. En Angular cada programa se puede ver como un árbol de módulos jerárquico. A partir de un módulo raíz se enlazan otros módulos en un proceso llamado importación.

## 2.1 Definición mediante decoradores
Antes de importar cualquier módulo hay que definirlo. En Angular los módulos de declaran como clases de TypeScript habitualmente vacías decoradas con una función especial. Es la función `@NgModule()` recibe un objeto como único argumento. Es en las propiedades de ese objeto dónde se configura el módulo. 

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

## 2.1 Importación de otros módulos
El módulo `App` también se conoce como módulo raíz porque de él surgen las demás ramas que conforman una aplicación. La asignación de los nodos hijos se realiza en la propiedad `imports:[]`, que es un array de punteros a otros módulos.

En la situación original el módulo principal depende un módulo pra realizar el enrutado (el `AppRoutingModule`) y de otro para la presentación en el navegador (el 'BrowserModule').

### 2.1.1 Imports de Angular e import de TypeScript
Si es la primera vez que ves código TypeScript te llamarán la atención las primeras líneas de cada fichero. En el `app.module.ts` son algo así:

```typescript
import { BrowserModule } from "@angular/platform-browser";
import { NgModule } from "@angular/core";
import { AppRoutingModule } from "./app-routing.module";
import { AppComponent } from "./app.component";
```

Estas sentencias de importación son propias del lenguaje y nada tienen que ver con Angular. En ellas se indica que este fichero importa el contenido de otros ficheros typescript. La importación es a través de la ruta física relativa al fichero actual o en el directorio `node_modules` si es código de terceros.

En general no tendrás que preocuparte de estas importaciones físicas, pues el *VSCode* y las extensiones esenciales se encargan de hacerlo automáticamente o te sugieren cómo hacerlo.  

## 2.2 Generación de módulos
Hasta ahora los módulos involucrados son librerías de terceros o se crearon mágicamente con la aplicación. Es hora de que crees tu primer módulo. Para eso usaremos otro comando del cli. En una ventana del terminal escribe:
```shell
ng g m lib/components
```
Esta es la sintaxis abreviada del comando [`ng generate`](https://github.com/angular/angular-cli/wiki/generate) el cual dispone de varios planos de construcción o *blueprints*. El que he usado aquí es el de `module` para la construcción de módulos.

El resultado es la creación del fichero `lib/components/components.module.ts` con la declaración y decoración del módulo `ComponentsModule`.
Este módulo te servirá de contenedor para guardar componentes como veremos más adelante. Por ahora hay asegurar que este módulo se engancha al raíz. Para ello dejaremos la línea de importación del módulo principal como sigue:

```typescript
imports: [AppRoutingModule, BrowserModule, ComponentsModule],
```

# 3. Componentes
ng g c lib/components/nav 
ng g c lib/components/main 
ng g c lib/components/footer --export 
## 3.1 Componentes privados
ng g c lib/components/nav/title --flat