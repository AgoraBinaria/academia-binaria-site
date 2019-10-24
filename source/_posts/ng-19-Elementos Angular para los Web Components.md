---
title: Elementos Angular para los Web Components
permalink: elementos-Angular-para-los-Web-Components
date: 2018-10-25 13:50:27
tags:
- Angular
- Angular8
- Angular2
- Elements
- WebComponents
- Tutorial
- Avanzado
categories:
- [Tutorial, Angular]
thumbnail: /css/images/angular-19_elements.png
---

![elementos-Angular-para-los-Web-Components](/images/tutorial-angular-19_elements.png)

La industria web vive un momento de esplendor y le crecen los _frameworks_ como hierbas primaverales. Pero el estándar HTML no se queda atrás y evoluciona hacia tecnologías potentes y genéricas. **Angular Elements** promueve la reutilización de código en distintos frameworks para que puedas usar tus componentes Angular en otros entornos.

Siendo como es Google una empresa web first, se esfuerzan en incorporar y adaptar de la mejor manera los estándares HTML a sus productos. Con el desarrollo de Angular siempre tuvieron la vista puesta en la tecnología de los **Web Components**. Buscando que los usos futuros del código se garantizasen más allá del framework de creación.

<!-- more -->

Partiendo del código tal como quedó en [Internacionalización y puesta en producción](../internacionalizacion-y-puesta-en-produccion-con-Angular/). Al finalizar tendrás una componentes que podrás utilizar fuera de Angular.

> Código asociado a este tutorial en _GitHub_: [AcademiaBinaria/angular-boss](https://github.com/AcademiaBinaria/angular-boss)

# 1. Componentes independientes del framework

> Hay lácteos que aguantan más que algunos frameworks.

Seguramente este no sea el caso de Angular, ni de otros frameworks de adopción masiva como React, Vue o Svelte. Todos ellos tienen un brillante presente y futuro al plazo que la tecnología pueda vislumbrar. Pero más temprano que tarde otra tecnología o paradigma disruptivo los desplazará. O al menos los obligará a cambiar tanto que sean irreconocibles.

Para entonces, y también mientras tanto, nuestro código será cautivo del framework en el que nació. Pero eso cambiará con los Web Components.

## 1.1 Origen y potencial

Los **Web Components** son independientes de los _frameworks._ Esta es la idea clave; se pueden desarrollar con el estándar pelado de JavaScript o con cualquier framework moderno. Pero lo fundamental es que no exigen nada especial para ejecutarse. Esto permite la interoperabilidad y también extiende la vida útil de tus creaciones.

### Usos posibles

Partiendo de dichas premisas es fácil vislumbrar todo el potencial y casos dónde aplicarlos. Por ejemplo:

- Librerías de diseño multiplataforma

- Migración paulatina de aplicaciones legacy

- Integración dinámica en grandes soluciones CMS

- Mejoras funcionales en aplicaciones server side

## 1.2 Estándares y tecnología

Bajo el término Web Components se esconden diversas tecnologías. No todas ellas están al mismo nivel de aceptación, e incluso alguna no ha visto la luz, pero este es un esbozo de lo que tenemos:

- Shadow DOM: Manipulación de un árbol en memoria antes de aplicar sus cambios al verdadero.
- HTML templates: Fragmentos de HTML que no se utilizan en la carga de la página, pero que se pueden instanciar más adelante
- ES Modules: Inclusión de documentos JS en forma de módulos estándar y ágil.
- **Custom elements:** son etiquetas HTML encapsuladas reutilizables para usar en páginas y aplicaciones web.

El estándar:

> Los **Custom Web Elements** sólo requieren HTML y JavaScript.


La tecnología:

> Angular Elements empaqueta tus componentes como **Custom Web Elements**.

# 2. Desarrollo y despliegue con Angular

## 2.1 Exponer los componentes
## 2.2 Compilación y despliegue

# 3. Consumo en HTML

## 3.1 Copiar
## 3.2 Importar


Ahora ya tienes una aplicación que se puede desplegar adaptada a las preferencias culturales de tus usuarios. Continúa tu formación avanzada para crear aplicaciones Angular siguiendo la serie del [tutorial avanzado de desarrollo con Angular](../tag/Avanzado/) y verás como aprendes a programar con Angular 8.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>