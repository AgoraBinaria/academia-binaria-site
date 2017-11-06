---
title: 'Angular2, la evolución de la plataforma'
tags:  
- Angular2
categories:
- Introducción 
permalink: angular2-primeras-impresiones
id: 24
updated: '2016-06-08 08:10:11'
date: 2016-05-06 15:57:50
---

Coincidiendo con la **ngConf 2016** acaba de presentarse la esperada *Release Candidate* de **[Angular 2](https://angular.io/)**. Nunca antes una versión de una herramienta para desarrolladores había causado tal expectación. Hace ya 18 meses que se había anunciado como una evolución rupturista con respecto a **AngularJS 1**. Y ahora esa ruptura se ha materializado. 

Los programadores somos muy conscientes de que **lo único estable es el cambio**. Asumimos, por experiencia, que los cambios en una tecnología son evoluciones graduales constantes. Pero esto no es así en **Angular2**, y lo sabemos desde su anuncio hace año y medio. Desde ese momento todos fuimos advertidos de que estábamos ante otra cosa. Algo nuevo que sólo comparte paternidad y nombre con la anterior versión.

<!-- more -->

## Un poco de historia

En septiembre de 2011 me tropecé con AngularJS buscando una alternativa a *Backbone* y *KnockOut*. Yo venía del mundo encorsetado del desarrollo para multinacionales basado en tecnologías *serias* como .Net y Java. Necesitaba un cambio, una apertura... pero con ciertas garantías. 

En esos tiempos el desarrollo web vivía su eclosión de la mano del HTML5. Era un mundo de *startups* (ahí estaba yo), de *earlyadopters* y de picaflores tecnológicos. Buscábamos **tecnologías simples, universales y de bajo coste**. Muy en la línea con el método empresarial *lean*. Pero el riesgo de escoger el framework perdedor era enorme, y cada pocos meses aparecían o desaparecían candidatos. 

Angular superó a sus contendientes por dos razones: una fue (rellena aquí con el argumentario técnico que más te satisfaga) y la otra fue **Google**. Un padrino así abre muchas puertas y da el punto de valor suficiente para alejarte de Oracle o Microsoft sin temor.

En enero de 2013 abandoné mi sueño de convertirme en el nuevo Zuckerberg. Volví al viejo sector servicios fundando una empresa de consultoría. Aprovechando los conocimientos y métodos ágiles aprendidos por el camino *startup*, ofrecimos desarrollos *low cost* para pequeñas empresas. Para complementar los servicios empecé a impartir cursos sobre estas tecnologías a otros desarrolladores. Al principio sólo acudían *freelances* y pequeñas consultoras.

En marzo de 2015 tuve los primeros contactos con **grandes empresas** tanto a nivel docente como consultor. Volví a un mundo familiar pero con una tecnología distinta. AngularJS había crecido de manera exponencial en número de desarrolladores. Pero ahora estaba jugando en las grandes ligas.
 
Y de repente aparecieron los problemas.

### Dos problemas

#### El lenguaje
JavaScript es dinámico, asíncrono y nada modular. Ideal para espíritus libres, pero una amenaza para equipos estrictos. Y necesitas control si vas a migrar grandes aplicaciones de negocio, ERPs , banca, servicios de administración pública. 

Tampoco se emocionaban los arquitectos de software con las **herramientas**. Como siempre todo empezó de manera sencilla: ficheros y editores de texto. Luego llegó la minificación, la combinación, el pre y pos procesado, las anotaciones, la documentación, las pruebas... Cada mes era mayor el arsenal de micro herramientas que había que orquestar.

#### La escalabilidad
Las aplicaciones empresariales son pesadas en lógica y datos. La falta de un cargador dinámico decente hacía muy difícil reducir el tiempo antes del primer impacto. Una vez lanzadas las aplicaciones iban razonablemente bien. 
Hasta que alguien creaba informes editables. Con miles de datos de los que preocuparse la técnica del **doble binding** saturaba los *watchers*. 

AngularJS moría de éxito. 

### Dos soluciones
#### El lenguaje: TypeScript
La lenta evolución de JavaScript parece haber salido de su hibernación. En el último año disfrutamos ya de las mejoras de **ES6 (ES2015)** y empezamos a probar **ES7 (2016)**. Pero no es suficiente.

Para grandes desarrollos, con miles de líneas de código, toda ayuda es poca. **TypeScript** asume todas las mejoras y propuestas del más avanzado JS estándar y además aporta **tipos**. Esa es la principal razón de su elección como lenguaje de cabecera en AngularJS2.

Al rededor de esa piedra angular crece el ecosistema de **herramientas**. Principalmente [VSCode](https://code.visualstudio.com/) que lo aprovecha ofreciendo *intellisense* y *refactoring* a la altura de los grandes.

La oferta se completa con *Interfaces*, *Generics* y otras novedades que harán las delicias de los programadores orientados a objetos. 

Cabe señalar que TypeScript no es ni mucho menos obligatorio. **Se puede desarrollar en ES5 y ES6** sin problemas. Pero la idea, los ejemplos, la documentación y los blogueros haremos que tu vida sea más fácil si escoges TypeScript.

#### La plataforma: renderización, observables, componentes, carga dinámica, SEO, apps, herramientas...
El **doble binding** era el plato estrella de AngularJS, pero salía caro. Así que se replanteó una solución que no requiriese un *pull* constante preguntando el estado de un montón de variables. Las respuesta vino de la mano de otra colaboración con los de Redmond: [ReactiveX](http://reactivex.io/). Este cambio será el primero al que te enfrentes si vienes de las conocidas versiones 1.

El mismo **patrón observable** se aplicó también a las comunicaciones HTTP. Nuestra aplicación es ahora un conjunto de *streams* que emiten eventos. Solo tenemos que suscribirnos y observar el estado cambiante de nuestro modelo. Resultado, aplicaciones hasta **cinco veces más rápidas**.

Los principios de encapsulación, modularidad y reutilización ya estaban en el ADN de AngularJS. Pero no era fácil implementarlos con las viejas directivas. Desde la versión 1.5 disponemos de **componentes**. Esta versión puente trae una simplificación y una guía sobre como estructurar aplicaciones. En AngularJS 2 van más allá, y tal como habían amenazado, matan al controlador y coronan al componente como nuevo rey del *front end*.

La **inyección de dependencias** fue uno de los grandes aciertos iniciales de AngularJS 1. Pero con el tiempo se convirtió en una rémora para crear grandes SPAs, pues requería disponer de toda la lógica, todo tu código, desde el primer segundo. Este problema se ha resuelto, aunque de forma demasiado tediosa. Hay margen de mejora en la actual implementación y espero que se vaya simplificando.

Uno de los pocos puntos débiles de AngularJS, y otros frameworks *client side*, era la dificultad para la indexación **SEO on site**. La solución normalmente pasaba por algún tipo de *prerenderizado* más o menos engorroso. Con la aparición de [Angular Universal](https://universal.angular.io/), podemos ejecutar **Angular en el servidor**. Esto abre las puertas al SEO y a la reducción de la espera en la primera visita.

Las aplicaciones híbridas han sido la solución *low cost* para que cualquier empresa accediese a los elitistas mercados de aplicaciones para móviles. Pero la solución no era perfecta. Con el [Angular Mobile Tookit](https://mobile.angular.io/) se ofrecen **aplicaciones progresivas** que compiten en rendimiento y funcionalidad con las nativas.

Poner en marcha y depurar una aplicación moderna de gran tamaño requiere herramientas a la altura. La oferta en este caso es total: 

- [Angular CLI](https://cli.angular.io/) una herramienta en línea de comandos para generar aplicaciones preconfiguradas
- [Augury](https://augury.angular.io/) para depurar y visualizar en el navegador el estado del programa
- [Protractor]() para los tests e2e
- [Guías de estilo](https://angular.io/styleguide) y analizadores de código... 

#### Una alternativa
[React](https://facebook.github.io/react/), la propuesta de Facebook para el desarrollo web, es una herramienta formidable. Rápido en ejecución, más cercano al estándar JS y mucho menos exigente en cuanto a herramientas. 

Parte con la enorme desventaja numérica que supone el más de un millón de programadores que ya conocen AngularJS y el ecosistema de librerías, componentes y material docente que ha surgido a su alrededor.

Será por esto último que aún no ha enganchado en la gran empresa. Por tanto aún no tira de ofertas de trabajo ni genera el ruido de la apuesta de Google. Eso sí, merece mucho la pena conocerlo, e incluso incorporarlo en algún caso junto a Angular 2.   

## Angular 2 no es plataforma para aficionados
El resumen es que AngularJS 1.x ha mutado **de framework a plataforma** en Angular 2. En este sentido estará cada vez más orientada a grandes desarrollos empresariales, y a salir definitivamente del navegador y ocupar servidores, escritorios y teléfonos. 

Seguro que requiere un gran esfuerzo de aprendizaje por su novedades radicales. Seguro que por el camino habrá giros por su bisoñez y amplitud. Pero seguro que estamos ante la **plataforma de desarrollo empresarial** con mejor futuro. De momento.

> En [esta presentación](https://docs.google.com/presentation/d/1VyMGTwiM7HmNMdSCXrrYpHIej-Y8ZwZKCan7KMqvdCA/edit?usp=sharing) tienes un resumen rápido de mis impresiones acerca de Angular2, y de cómo empezar a programar aplicaciones universales.