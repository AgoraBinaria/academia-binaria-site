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

El _ecosistema_ de Angular está repleto de librerías para desarrolladores profesionales. Algunas hacen uso de los **schematics**, y entre ellas destaca [Angular Material](https://material.angular.io/). Esta implementación de la casa de la guía de diseño _Material Design_ de Google usa las capacidades de estas plantillas del CLI que permiten agregar librerías y generar código.

Un programador Angular debe **dominar el CLI** y debe conocer los beneficios que aporta un repositorio de multi-proyecto. Hay escenarios complejos muy adecuaos para estos _mono-repos_. Pero con el CLI es muy sencillo crear y usar nuevas aplicaciones dentro de un repositorio.

<!-- more -->

Partiendo de la aplicación tal cómo quedó en [Formularios reactivos con Angular](../formularios-reactivos-con-Angular/). Al finalizar tendrás, en el mismo repositorio, una nueva aplicación con la apariencia y usabilidad de _Material design_.

> Código asociado a este artículo en _GitHub_: [AcademiaBinaria/angular-board/projects/schemat/](https://github.com/AcademiaBinaria/angular-board/tree/master/projects/schemat)
>
> > Tienes una versión desplegada operativa para probar [Angular Board](https://academiabinaria.github.io/angular-board/)

# 1. Repositorio multi-proyecto

El primer comando que se usa al empezar con angular es `ng new mi-aplicacion`. Desde ese momento tu mundo es la carpeta `/src` en la que se genera el código y en la que vas a desarrollar.

Pero con el tiempo, ciertos proyectos crecen hay que dividirlos. O quizás surjan proyectos hermanos. Angular CLI permite disponer de más de un proyecto compartiendo repositorio y configuración.

## 1.1 Carpetas src y projects

Dado un repositorio inicial, para agregar una nueva aplicación usaremos el viejo comando _generate_. Por ejemplo voy a crear una aplicación en la que usar las capacidades de los **schematics** y de **material**; la llamaré _schemat_

```console
ng g application schemat --routing
```

Esta aplicación comparte la configuración básica de `angular.json` y las dependencias y scripts de `package.json`. Su código específico va en la carpeta `projects` destinada a los nuevos proyectos generados tras haber creado el repo inicial.


## 1.2 Compilación multi - proyecto

A partir de ahora cada comando del CLI debería ir asociado a un proyecto concreto. Digo debería porque an `angular.json` puedes establecer un proyecto por defecto, que si no dices lo contrario será el inicial.

Pero, es buena práctica crear scripts específicos en el `package.json` para iniciar y compilar cada proyecto.

```
"start:schemat": "ng serve schemat --aot -o --port 4271",
"build:prod:schemat": "ng build schemat --prod",
```

# 2. Instalación y configuración de Material

Por muchas funcionalidades que aporte un framework como Angular, siempre necesitaremos echar mano de alguna librería de terceros. Normalmente eso implica instalarla con _npm_, importar sus módulos en Angular y en ocasiones alguna configuración extra.

Pero algunos proyectos ha adoptado la librería _schematics_ para facilitar la adopción de sus librerías. Es el caso de **Angular Material**.

## 2.1 Agregar dependencias con schematics

Para agregar _Material_ un proyecto basta con usar el comando `add` del CLI.

```console
ng add @angular/material --project=schemat
```

Esto instalará y anotará la dependencia de `@angular/material` y otros paquetes necesarios. Pero además los registrará en `AppModule` y lo configurará.

## 2.2 Estilos, iconos y temas básicos

En el `index.html` se insertarán los enlaces a las hojas de estilos con fuentes e iconos. En el `styles.css` podremos importar el tema de _Material_ que nos guste.

```css
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
