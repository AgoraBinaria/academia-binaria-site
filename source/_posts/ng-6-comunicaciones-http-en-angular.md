---
title: Comunicaciones http en Angular
permalink: comunicaciones-http-en-Angular
date: 2017-12-18 11:06:00
tags:  
- Angular
- Angular5
- http
- Tutorial
categories:
- [Tutorial, Angular] 
thumbnail: /css/images/angular-5_6_http.png
---

![Tutorial Angular5 6-http](/images/tutorial-angular-5_6_http.png)

La presentación, la lógica y el manejo de datos son tres capas de abstracción que usamos los programadores para mantener organizado nuestro código. En Angular, la presentación es cosa de los componentes. **La lógica y los datos tienen su lugar en servicios compartidos**.

Para que los componentes consuman los servicios de forma controlada tenemos _inyectables_ en la librería `@angular/core` con los que realizar **la inyección de dependencias**.

<!-- more -->

Partiendo de la aplicación tal cómo quedó en [Servicios inyectables en Angular](../servicios-inyectables-en-Angular/). Al finalizar tendrás una aplicación que comunica componentes entre páginas, reparte responsabilidades y gestiona claramente sus dependencias.

> Código asociado a este artículo en _GitHub_: [AcademiaBinaria/angular5/6-http](https://github.com/AcademiaBinaria/angular5/tree/master/6-http/cash-flow)

# 1. Servicios

Este caso de uso mantiene los datos en memoria, lo cual es muy poco fiable y sólo debe usarse con información muy volatil. Sigue esta serie para añadirle [vigilancia y seguridad en Angular](../categories/Tutorial/Angular/) mientras aprendes a programar con Angular5.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>
