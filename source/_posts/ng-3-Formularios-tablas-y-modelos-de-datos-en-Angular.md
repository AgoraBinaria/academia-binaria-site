---
title: Formularios, tablas y modelos de datos en Angular
permalink: formularios-tablas-y-modelos-de-datos-en-angular
date: 2017-11-15 10:17:37
tags:  
- Angular
- Angular5
- Forms
- Tutorial
categories:
- [Tutorial, Angular] 
thumbnail: /css/images/angular-5_3_data.jpg
---
![Tutorial Angular5 3-Data](/images/tutorial-angular-5_3_data.png)

Las **aplicaciones Angular son excelentes para el tratamiento de datos** en el navegador. La recogida de información mediante formularios y la presentación de páginas dinámicas fue su razón de ser.

Vamos a ver cómo la librería `@angular/forms` enlaza **las vistas, los controladores y los modelos** y cómo se hace la presentación de datos en **listas y tablas**.

<!-- more -->

Partiendo de la aplicación tal cómo quedó en [Páginas y rutas Angular SPA](../paginas-y-rutas-angular-spa/). Al finalizar tendrás una aplicación que recoge y presenta datos.

>Código asociado a este artículo en *GitHub*: [AcademiaBinaria/angular5/3-data](https://github.com/AcademiaBinaria/angular5/tree/master/3-data/cash-flow) 


# 1. Formularios
Los formularios son el punto de entrada de información a nuestros sistemas. Llevan con nosotros desde el inicio de la propia informática. Y se comen una buena parte del tiempo de programación. En *Angular* se ha prestado una importante atención a ellos facilitando su desarrollo, desde pantallas simples a complejos procesos.

## 1.1 El Binding
La clave para entender cómo funciona *Angular* está en el concepto de enlace entre elementos html de las vistas y propiedades de modelos de datos, el llamado `binding`.

Ya hemos visto ejemplos sencillos de binding en este tutorial. Por ejemplo en el fichero `title.component.ts` tenemos en su vista html la directiva `{{ title }}`. Esas dobles llaves encierran expresiones que se evaluarán en tiempo de ejecución. La llamamos **directiva de interpolación** y es la manera más cómoda y usual de mostrar contenido dinámico en Angular.

```typescript
@Component({
  selector: "cf-title",
  template: `
    <a routerLink="/">{{ title }}</a>
    <a routerLink="/operations">Operations</a>`
})
export class TitleComponent implements OnInit {
  title = "Cash Flow";
  constructor() {}
  ngOnInit() {}
}
```

La expresión interna hace referencia a variables que se obtienen de las propiedades de la clase controladora del componente. En este caso `TitleComponent.title`, con su valor *Cash Flow* en ejecución.

> To be continued... Work in progress...  