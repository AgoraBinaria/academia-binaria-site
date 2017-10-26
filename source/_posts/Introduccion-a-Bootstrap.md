---
title: Introducción a Bootstrap
tags: 
- Bootstrap
categories:
- Introducción 
permalink: introduccion-a-bootstrap
id: 5
updated: '2015-11-12 15:56:16'
date: 2015-11-05 17:44:55
---

Bootstrap es un conjunto de archivos **HTML**, **CSS** y **Javascript** bien definidos y exhaustivamente testados que componen una sólida estructura sobre la cual construir tu proyecto web o aplicación.

### ¿Por qué debo utilizar Bootstrap en mi proyecto?

Empezar un proyecto web desde cero puede ser laborioso y te puede llevar bastante tiempo mientras vas construyendo los diferentes componentes y testándolos en los diversos navegadores.

Bootstrap te provee de una **biblioteca de componentes reusables** que puedes usar en cada proyecto, desde un sistema de rejilla o “grid”, una barra de navegación, botones o estilos tipográficos, todos ellos preparados para funcionar en todos los navegadores y adaptarse a todos los tipos de pantalla, tanto de móvil, como tablet o escritorio.

Con Bootstrap te ahorras todo el trabajo inicial y además tienes un código revisado y testado que además es **mantenible y escalable**, una sólida estructura sobre la cual construir tu proyecto.

### Práctica común

Hoy en día su uso se encuentra **muy extendido**, y no es extraño encontrarse ofertas de trabajo en las cuales además de conocimientos de HTML y CSS y Javascript se exige el dominio de Bootstrap. La **suave curva de aprendizaje** hace que sea una herramienta sencilla de usar por todos los miembros de un equipo de desarrollo, lo que incrementa la productividad.

### ¿Por dónde empezar?

Para descargar la última versión debes acudir a la página http://getbootstrap.com/  y pulsar el botón **“Download Bootstrap”** una vez hecho esto te dirigirá a la sección **“Download”** de la página. Hay diferentes maneras de integrar Bootstrap en tu proyecto, en este artículo hablaremos de la primera de ellas que es descargarnos la carpeta comprimida.

En el apartado **“Bootstrap”** de la sección **“Download”**  te encontrarás otra vez el botón de **“Download Bootstrap”**, esta vez al clicarlo se descargará una carpeta comprimida que se compone de:

-	Una carpeta **”css”**: Incluye todos los estilos predefinidos por Bootstrap.
-	Una carpeta **“fonts”**: Incluye una fuente de iconos svg listos para usar.
-	Una carpeta **“js”**: Con todo el javascript necesario para que los componentes y efectos de Bootstrap funcionen.

Una práctica común es empezar el proyecto con estas carpetas e incluir a través de tu HTML los archivos que vas a utilizar. Normalmente **bootstrap.min.css** y **bootstrap.min.js**. Además deberás **incluir JQuery** en tu proyecto para el correcto funcionamiento de los plugins.

Una vez hecho esto te debería quedar un HTML inicial parecido a esto:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <!-- Los 3 metatags de arriba deben ir al principio del head -->
    <title>Mi primer tema con Bootstrap</title>

    <!-- Incluye el css de Bootstrap -->
    <link href="css/bootstrap.min.css" rel="stylesheet">
    <!-- Incluye nuestro propio CSS -->
    <link href="css/styles.css" rel="stylesheet">


    <!-- Este código es necesario si queremos dar soporte a IE8 -->
    <!-- WARNING: Respond.js doesn't work if you view the page via file:// -->
    <!--[if lt IE 9]>
      <script src="https://oss.maxcdn.com/html5shiv/3.7.2/html5shiv.min.js"></script>
      <script src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
    <![endif]-->
  </head>
  <body>
    <h1>Hola!</h1>

    <!-- Incluye jQuery -->
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.11.3/jquery.min.js"></script>
    <!-- Incluye el JS de Bootstrap -->
    <script src="js/bootstrap.min.js"></script>
  </body>
</html>

```

Con esto ya estarías listo para empezar tu proyecto con Bootstrap y, con la ayuda de la documentación de Bootstrap, ir incluyendo los módulos y clases que necesites.

Cómo norma general **nunca modifiques los archivos originales de Bootstrap**, para añadir tus estilos propios o modificar los de bootstrap.css lo mejor es que crees una hoja de estilos independiente de bootstrap.css, lo mismo para el archivo bootstrap.js, así para actualizar tu versión de Bootstrap simplemente tendrás que descargarte la última versión y sustituir los archivos bootstrap.min.css y bootstrap.min.js por los de la nueva versión.

En siguientes artículos profundizaremos un poco mas sobre las diferentes opciones que tienes para incluir Bootstrap en tu proyecto.
