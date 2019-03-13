---
title: Material Design y CLI de Angular
permalink: Material-Design-y-CLI-de-Angular
date: 2019-03-14 18:59:27
tags:
- Angular
- Angular7
- Angular2
- material
- Tutorial
- Introducción
categories:
- [Tutorial, Angular]
thumbnail: /css/images/angular-9_material.png
---

![Material-Design-y-CLI-de-Angular](/images/tutorial-angular-9_material.png)

El _ecosistema_ de angular está repleto de librerías para desarrolladores profesionales. Algunas hacen uso de los **schematics**, entre las que destaca  *Material Design*. Esta implementación de la casa de la guía de diseño de Google usa las capacidades de estas plantillas del CLI que permiten agregar librerías y generar código.

Un programador Angular debe dominar el CLI y debe conocer los beneficios que aporta un repositorio de multi-proyecto. Hay escenarios más adecuados que otros para estos nono-repos. Pero con el CLI es muy sencillo crear y usar nuevas aplicaciones dentro de un repositorio.

<!-- more -->

Partiendo de la aplicación tal cómo quedó en [Formularios reactivos con Angular](../formularios-reactivos-con-Angular/). Al finalizar tendrás, en el mismo repositorio, una nueva aplicación con la apariencia y usabilidad de _Material design_.

> Código asociado a este artículo en _GitHub_: [AcademiaBinaria/angular-board/projects/schemat/](https://github.com/AcademiaBinaria/angular-board/tree/master/projects/schemat)
>
> > Tienes una versión desplegada operativa para probar [Angular Board](https://academiabinaria.github.io/angular-board/)

# 1. Repositorio multi-proyecto

## 1.1 Carpetas src y projects

CLI

```console
ng g application schemat --routing
```

angular.json

```
"es5BrowserSupport": true
```

## 1.2 Compilación multi - proyecto

package.json

```
"start:schemat": "ng serve schemat --aot -o --port 4271",
"build:prod:schemat": "ng build schemat --prod",
```

# 2. Instalación y configuración de Material

## 2.1 Agregar dependencias con schematics

```console
ng add @angular/material --project=schemat
```

## 2.2 Estilos, iconos y temas básicos

index.html

app.module.ts

styles.css

```
@import '~@angular/material/prebuilt-themes/indigo-pink.css';
```

# 3. Componentes básicos

## 3.1 Navegación y layout

### 3.1.1 Navegación

``` console
ng g @angular/material:nav shell --project=schemat
```

app.component.html

```html
<app-shell></app-shell>
```

### 3.1.2 Dashboard

``` console
ng g @angular/material:dashboard home --project=schemat
```

app-routing

```typescript
const routes: Routes = [
  {
    path: '',
    component: HomeComponent
  }
];
```

shell.component.html
```html
<a mat-list-item [routerLink]="['/']">Home</a>
<!-- Add Content Here -->
<router-outlet></router-outlet>
```

## 3.2 Componentes básicos

### 3.2.1 Formularios

``` console
ng g @angular/material:address-form contact --project=schemat
```

app-routing

```typescript
const routes: Routes = [
  {
    path: 'new-contact',
    component: ContactComponent
  }
];
```

shell.component.html
```html
<a mat-list-item [routerLink]="['/new-contact']">New Contact</a>
```

### 3.2.2 Tablas

``` console
ng g @angular/material:table elements --project=schemat
```

app-routing

```typescript
const routes: Routes = [
  {
    path: 'elements-list',
    component: ElementsComponent
  }
];
```

shell.component.html
```html
<a mat-list-item [routerLink]="['/elements-list']">Elements List</a>
```

elements.component.html

```html
<mat-paginator #paginator
    [length]="dataSource.data.length"
    [pageIndex]="0"
    [pageSize]="5"
    [pageSizeOptions]="[5, 10, 15, 20]">
</mat-paginator>
```

---

### 3.2.3 Árboles

``` console
ng g @angular/material:tree source --project=schemat
```

app-routing

```typescript
const routes: Routes = [
  {
    path: 'source-tree',
    component: SourceComponent
  }
];
```

shell.component.html
```html
<a mat-list-item [routerLink]="['/source-tree']">Source Tree</a>
```


Con este conocimiento finalizas tu [introducción a Angular](../tag/Introduccion/).  En el tutorial avanzado aprenderás más cosas para programar con Angular 7.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>
