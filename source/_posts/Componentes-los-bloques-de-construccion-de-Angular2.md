---
title: 'Componentes, los bloques de construcción de Angular2'
tags:  
- Angular2
- CLI
- Tutorial
categories:
- Introducción 
permalink: componentes-los-bloques-de-construccion-de-angular-2
id: 26
updated: 2016/11/07 18:39:26
date: 2016/05/23 14:07:27
---

> ACTUALIZACIÓN: para una versión más reciente del contenido visita la página [Base para una aplicación Angular](../base-aplicacion-angular/)

---

>Código asociado en GitHub: [angular2/2-modulos/](https://github.com/AcademiaBinaria/angular2/tree/master/2-modulos) 

Los componentes son los bloques de construcción de la web moderna. En Angular 2 sustituyen al viejo par Vista-Controlador, el cual ya era opcional en las últimas versiones 1.x Ahora **el componente es el rey**.

Las aplicaciones en Angular2 se desarrollan como **árboles de componentes**. Estos árboles pueden llegar a ser muy frondosos y conviene dirigir su crecimiento agrupándolos en **módulos** para no perderse. Yo procuro estructurarlo en niveles para una mejor comprensión. En cada nivel se crea un módulo y dentro de él se declaran los componentes.

* 1- Nivel **Raíz**:
Toda aplicación parte de un componente raíz. Suele recibir el nombre de la aplicación desarrollada y el sufijo `App` o simplemente `App`. 
* 2- Nivel **Troncal**:
Generalmente dos o tres componentes troncales para la estructura de las páginas. Es común el patrón *Navegador-Contenedor*, con algún elemento auxiliar para ayudas, mensajes, menús complejos... 
* 3- Nivel de **Ramas**:
En este símil, las ramas equivalen a rutas o vistas de la aplicación. En un *SPA* cada ruta tiene una vista asociada que se carga dentro del componente troncal contenedor 
* 4- Nivel de **Hojas**:
Cada una de las vistas está a su vez formada por múltiples componentes de negocio a modo de hojas. 

![Árbol de componentes en una aplicación Angular 2](/images/ng2-Arbol-de-componentes.jpg)

>Con la salvedad de que muchos de estos componentes los puedes reutilizar en distintas vistas.

<!-- more -->

## Módulos

Los árboles de componentes pueden ocultarnos fácilmente el bosque de nuestra aplicación. **Los módulos son agrupaciones de componentes.** Nos ayudan a mantener un orden y a encapsular funcionalidad para crear aplicaciones desacopladas con bloques re-utilizables.

> No confundir con los módulos de *JavaScript ES6* o de T*ypeScript*. En estos casos se les llama módulos a los ficheros de código que exportan funcionalidad. 

Podemos imaginar un módulo como una fábrica de funcionalidad.
1. **Importa** componentes que otros módulos exportan.
2. **Declara** los componentes que el mismo fabrica.  
3. **Exporta** algunos de estos componentes, para que los consuman otros módulos.


### Anatomía de un Componente
En el artículo de [bienvenida a Angular 2](http://academia-binaria.com/hola-mundo-en-angular-2/) teníamos una aplicación de un sólo módulo con un sólo componente. Y nos sirvió para ver su **estructura**: plantilla, decorador y clase.

![Estructura interna de un componente Angular 2](/images/ng2Component--1-.jpg)

La **plantilla** en HTML y la **clase** en JS equivalen a las antiguas vistas y controladores. La **metadata** une ambos mundos y registra el componente para que interacciones con el resto del mundo Angular. 

>La comunicación de datos entre la plantilla y el componente se realiza siguiendo un remozado workflow de propiedades y eventos. El nuevo *data-binding* de Angular2 merece estudio en detalle pues su sintaxis ha cambiado para poder estar a la altura del rendimiento exigido. 




### Generación de módulos y componentes con angular-cli

Sobre la base de ese 'Hola Mundo' vamos construir una mini aplicación muy sencilla para guardar movimientos económicos. El *To Do List* de los ingresos y gastos. 

Empezaré creando **otro módulo con su componente para ser integrado** en el componente raíz del módulo raíz. Por ahora será un componente de negocio vacío: el componente `movimientos`. Puedes escribir a mano cada nuevo módulo o componente, pero si usas [Angular CLI](https://cli.angular.io/) lo tendrás generado con un sólo comando:

```bash 
ng g m movimientos
``` 

Verás que se ha creado una carpeta llamada `movimientos` con una estructura que pronto te será muy familiar. Para empezar un archivo para el nuevo módulo, el `movimientos-module.ts`. Después una serie de ficheros para crear su componente principal. Un fichero `movimientos-component.ts` para la clase controladora y el decorador con la *metadata*, y otro fichero `movimientos-component.html` para la plantilla.

Reproduzco ahora su contenido básico. Primero la plantilla HTML

```html
<p>
  movimientos works!
</p>
```
Y ahora la definición del componente en *TypeScript*. El cual no sabe en qué módulo acabará.

```javascript
import { Component, OnInit } from '@angular/core';
// decoración con metadata para el componente
@Component({
  selector: 'app-movimientos', // ojo al prefijo, por defecto app
  templateUrl: './movimientos.component.html', // podrían ser inline
  styleUrls: ['./movimientos.component.css'] // podrían ser inline
})
export class MovimientosComponent implements OnInit {
  constructor() { }
  ngOnInit() {
  }
}
```

Para que este componente sea conocido ha de estar al menos declarado y exportado en algún módulo. En este caso aparece en el fichero `movimientos-module.ts`

```javascript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
/** Importa un componente que declara y que luego exporta */
import { MovimientosComponent } from './movimientos.component';

@NgModule({
  imports: [
    CommonModule
  ],// dependencias de otros módulos
  declarations: [
    MovimientosComponent
  ],// Componente que el módulo mismo declara
  exports: [
    MovimientosComponent
  ] // exporta los componentes importables desde otros módulos
})
export class MovimientosModule { }
```

### Consumo desde un componente padre
El componente `movimientos` está creado pero nadie lo usa. Para darle utilidad hay que consumirlo. Vamos tocar los ficheros 'app.module.ts' y 'app.component.html' Son siempre estos tres sencillos pasos:

* 1- Importar el módulo que lo exporta
```javascript
// 1 importación del código del módulo funcional
import { MovimientosModule } from './movimientos/movimientos.module';
```
* 2- Registro en el array de importaciones del módulo raíz
```javascript
...
imports:[..., MovimientosModule] // 2 registro del modulo importado con todo lo que exporta
...
```
* 3- Uso del componente como un elemento html en la plantilla del padre
```html
<h1>
  <!--enlace con propiedades del componente-->
  {{title}}
</h1>
<!-- 3 los componentes personalizados se usan como elementos estándar en html-->
<!--componente movimientos-->
<app-movimientos></app-movimientos>
```


Esta manera de encapsular componentes unos dentro de otros permite crear **grandes aplicaciones de tamaño empresarial** sin sacrificar la limpieza del código. Cada componente debe diseñarse de forma que resuelva un problema de negocio concreto, por tanto manejable. Y si puede ser **reutilizable** mucho mejor.

La agrupación de componentes en módulos ayuda a mantener la aplicación organizada.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>

---

### ACTUALIZACIÓN

Para una versión más reciente del contenido visita la página [Base para una aplicación Angular](../base-aplicacion-angular/)