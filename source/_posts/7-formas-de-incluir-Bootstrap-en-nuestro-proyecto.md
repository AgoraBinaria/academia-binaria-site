---
title: 7 formas de incluir Bootstrap en nuestro proyecto
tags:  
- Bootstrap
categories:
- Introducción 
permalink: formas
id: 6
updated: 2015/11/12 16:55:15
date: 2015/11/12 15:40:12
---

### 1- Descargando los archivos

Haz clic en el botón **"Download Bootstrap"** y se descargará una carpeta comprimida, al descomprimirla te encontrarás las versiones compiladas y minificadas de Bootstrap así como los iconos que utiliza:

```
bootstrap/
├── css/
│   ├── bootstrap.css
│   ├── bootstrap.css.map
│   ├── bootstrap.min.css
│   ├── bootstrap.min.css.map
│   ├── bootstrap-theme.css
│   ├── bootstrap-theme.css.map
│   ├── bootstrap-theme.min.css
│   └── bootstrap-theme.min.css.map
├── js/
│   ├── bootstrap.js
│   └── bootstrap.min.js
└── fonts/
    ├── glyphicons-halflings-regular.eot
    ├── glyphicons-halflings-regular.svg
    ├── glyphicons-halflings-regular.ttf
    ├── glyphicons-halflings-regular.woff
    └── glyphicons-halflings-regular.woff2

```



### 2- Para usuarios de GitHub

Bootstrap es un proyecto en código abierto así que puedes **clonar** o hacer **fork** desde [GitHub](https://github.com/twbs/bootstrap)

### 3- Código fuente original

Lo obtienes al hacer clic en el botón **"Download source"**. Lo que obtienes son los archivos **LESS** y **Javascript** originales. Esta opción requiere **Grunt** y **Node.js** y se estructura dentro de la siguiente manera:

```
bootstrap/
├── less/
├── js/
├── fonts/
├── dist/
│   ├── css/
│   ├── js/
│   └── fonts/
└── docs/
    └── examples/
```
### 4- Fans de Bower

Si tenemos **Git** y **Bower** instalado simplemente debes teclear en tu terminal:

```
$ bower install bootstrap
```

Y el pajarito hará todo el trabajo.

### 5- NPM Install

Tan fácil como con Bower solo que esta vez debes teclear en tu terminal:

```
$ npm install bootstrap
```

### 6- Versión en SASS

A pesar de que **Bootstrap 3 fue escrito en LESS** (aspecto que cambiará en la versión 4, que será en SASS), existe una versión para SASS que puedes descargar en el botón **"Download SASS"** para facilitar su inclusión en proyectos en **Rails**, **Compass** o solo **SASS**.

### 7- CDN

Por último, si no quieres descargarte nada puedes utilizar los links del proveedor de **CDN** y referenciarlos en tu HTML:

```html
<!-- CSS -->
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/css/bootstrap.min.css" integrity="sha512-dTfge/zgoMYpP7QbHy4gWMEGsbsdZeCXz7irItjcC3sPUFtf0kuFbDz/ixG7ArTxmDjLXDmezHubeNikyKGVyQ==" crossorigin="anonymous">

<!-- Tema opcional -->
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/css/bootstrap-theme.min.css" integrity="sha384-aUGj/X2zp5rLCbBxumKTCw2Z50WgIr1vs/PFN4praOTvYXWlVyh2UtNUU0KAUhAX" crossorigin="anonymous">

<!-- JavaScript -->
<script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/js/bootstrap.min.js" integrity="sha512-K1qjQ+NcF2TYO/eI3M6v8EiNYZfA95pQumfvcVrTHtwQVDG+aHRqLi/ETn2uB+1JqwYqVG3LIvdm9lj6imS/pQ==" crossorigin="anonymous"></script>
```

Esta es la manera más fácil pero no podrás trabajar sin conexión a internet.




