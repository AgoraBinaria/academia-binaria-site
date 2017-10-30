---
title: Enrutador de componentes con Angular2 el nuevo SPA
tags:  
- Angular2
categories:
- Introducción 
permalink: enrutado-con-angular2-el-nuevo-spa
id: 15
updated: 2016/12/07 18:23:02
date: 2016/06/02 08:52:04
---

> Código asociado en [angular2/6-routing/](https://github.com/AcademiaBinaria/angular2/tree/master/6-routing)

La capacidad de gestionar las **rutas en el cliente** es una de las grandes ventajas de *AngularJS*. En la versión 1 nos ofrecían una solución demasiado simple que obligaba a usar librerías de terceros como la famosa [ui-router](https://github.com/angular-ui/ui-router). Hemos esperado años la promesa de un nuevo *enrutador* compatible con las versiones 1 y 2. El resultado es **@angular/router**.

De todo *Angular2*, este es el componente que más ha cambiado durante la fase *Release Candidate*. Actualmente confiamos en que las pequeñas dudas se resuelvan definitivamente y podamos relajarnos creando modernos desarrollos **SPA**.


## Módulo de enrutado en base a componentes

Las aplicaciones Angular2 son [árboles de módulos](http://academia-binaria.com/componentes-los-bloques-de-construccion-de-angular-2/). Al menos el módulo raíz `AppModule` y cuantos módulos funcionales nos hagan falta. Utilizando `angular-cli` podemos crear módulos con la capacidad de enrutado generada en... un módulo específico.  

A ver si desbrozamos algo este bosque de módulos.

```javascript
ng new mi-aplicacion --routing true
```

Tanto para la raíz como para las ramas funcionales se creará un fichero. En la raíz será llamado `app-routing.module.ts` con un contenido como este: 

```javascript
/** Módulos de enrutado de Angular2 */
import { RouterModule, Routes } from '@angular/router';

import { NgModule } from '@angular/core';

// Array con las rutas de este módulo. Ninguna funcional.
const routes: Routes = [
  { path: '', redirectTo: '' },
  { path: 'inicio', redirectTo: '' },
  { path: '**', redirectTo: '', pathMatch: 'full' }
];

@NgModule({
  imports: [
    RouterModule.forRoot(routes) // configuración para el módulo raíz
  ],
  exports: [
    RouterModule // se importará desde el módulo padre
  ]
})
export class AppRoutingModule { }
```

Este **módulo** de un único fichero sirve **para definir las rutas de otro módulo padre** asociado, `app.module.ts`, el cual quedará más o menos así:

```javascript
// importación de módulo de enrutado asociado
import { AppRoutingModule } from './app-routing.module';
// importación de otros módulos de funcionalidad
import { HomeModule } from './home/home.module';
// decorador que define un módulo
@NgModule({
  declarations: [ AppComponent ], 
  imports: [
    BrowserModule,
    FormsModule,
    HttpModule,
    HomeModule, // el módulo funcional para la 'página home'
    AppRoutingModule // el módulo de rutas ya configurado
  ], 
  providers: [] 
  bootstrap: [ AppComponent ] 
})
export class AppModule { }
```

## Módulos funcionales
El módulo raíz es fundamental pero nada operativo. Toda el que sea de interés para los usuarios estará en módulos funcionales. En las aplicaciones SPA es una buena práctica crear **un módulo por cada ruta** principal. Incluida la página home. Por ejemplo usando el siguiente comando:

```javascript
ng generate module home --routing true
```
Aparece un fichero llamado `home-routing-module.ts`. Este módulo se debe configurar para que gestione sus propias rutas.

```javascript
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';
/** Componente enrutable */
import { HomeComponent } from './home.component';
/** Rutas asociadas a componentes */
const routes: Routes = [
  { path: '', component: HomeComponent },
];
/** array de componentes enrutables */
export const routableComponents = [
  HomeComponent
];
@NgModule({
  imports: [
    RouterModule.forChild(routes) // Para módulo funcional
  ],
  exports: [
    RouterModule // listo para importarlo en HomeModule
  ]
})
export class HomeRoutingModule { }
```

Se crean dos arrays relacionados. EL principal, `routes`, contendrá las **rutas pareadas con sus componentes** respectivos. 
Los cuales también se exportan directamente en la variable `routableComponents`. Esto se hace por comodidad. Para no tener que volver a importarlos en la declaración del módulo padre, que quedará así:

```javascript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
/** Importación de las rutas y sus componentes */
import { HomeRoutingModule, routableComponents } from './home-routing.module';
@NgModule({
  imports: [
    CommonModule,
    HomeRoutingModule // el módulo que sabe enrutar
  ],
  declarations: [
    routableComponents // los componentes de enrutado
  ]
})
export class HomeModule { }
```

### Carga de componentes según la ruta
Todo este trabajo de configuración se materializa en las **vistas**. Necesitaremos un lugar dónde inyectar el **componente de turno asociado a la ruta** actual. Presentamos el `router-outlet`. Y para que la aplicación pueda llevarnos de un lugar a otro usaremos  la directiva `routerLink`. 

De forma que la plantilla raíz `app.component.html` quedará normalmente así:

```html
<!--menú de navegación, sin href-->
<nav>
  <a [routerLink]="['/']">Inicio</a>
  <a [routerLink]="['/login']">Log In</a>
  <a [routerLink]="['/contacto']">Contacto</a>
</nav>
<!--Este componente nativo hace que el enrutador cargue una página dinámicamente-->
<router-outlet></router-outlet>
```


## Rutas hijas y con parámetros
De poco vale un enrutador que sólo atienda a los casos sencillos. Las aplicaciones profesionales plantean mayores retos. 
### Rutas hijas
Un ejemplo son las **rutas anidadas**, aquí llamadas rutas hijas. Son aquellas en las que una parte de la visualización es común y otra depende de la ruta concreta.

Veamos un ejemplo dónde se pretende dar de alta y mostrar una lista de elementos. La base de cualquier *CRUD*. Así quedaría el fichero `movimientos-routing.module.ts`.
 
```javascript
/** Importación de los componentes enrutables */
import { MovimientosComponent } from './movimientos.component';
import { ListaComponent } from './lista/lista.component';
import { NuevoComponent } from './nuevo/nuevo.component';

const routes: Routes = [
  {
    path: 'movimientos',
    component: MovimientosComponent,
    children: [ // rutas hijas, se verán dentro del componente padre
      {
        path: 'nuevo', // la ruta real es movimientos/nuevo
        component: NuevoComponent
      },
      {
        path: 'lista',
        component: ListaComponent
      }
    ]
  }
];
export const routableComponents = [
  NuevoComponent,
  ListaComponent,
  MovimientosComponent
]
```
Este mini-módulo es muy denso. Contrasta con la simplicidad del `movimientos.module.ts` que reduce su responsabilidad. Sólo tiene que importar los módulos adecuados.

```javascript
import { FormsModule } from '@angular/forms';
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
/** Módulo de enrutado y componentes */
import { MovimientosRoutingModule, routableComponents } from './movimientos-routing.module';

@NgModule({
  imports: [
    CommonModule,
    FormsModule,
    MovimientosRoutingModule
  ], // dependencias de otros módulos, especialmente el de enrutado
  declarations: [
    routableComponents
  ], // Los componentes vienen del fichero movimientos-routing.module.ts
  exports: [

  ], // no necesita exportar nada
})
```

Vemos que la idea es que los ficheros de definición de módulos funcionales **deleguen toda la lógica** posible a su propio módulo de enrutado.

Mientras tanto en la vista... Habíamos dejado la *template* del componente raíz con un `router-outlet`. Será ahí dónde se cargue el `MovimientosComponent` cuando se active la ruta *'/movimientos'*. Pero, ¿y las rutas hijas?, ¿qué se carga y dónde se carga cuando se activa la ruta *'/movimientos/nuevo'* o la ruta *'/movimientos/lista'* ?.

Echemos un vistazo a `movimientos.component.html` para comprobar que dispone de su propio `router-outlet`. 

```html
<nav>
   <a routerLink="/movimientos/nuevo" >Nuevo Movimiento</a>
   <a routerLink="/movimientos/lista" >Lista de Movimientos</a>
</nav>
<router-outlet></router-outlet>
```

Será en este elemento donde se inyecten los componentes asociados a las rutas hijas. Este *anidamiento* permite hacer aplicaciones modulares y acceder a vistas específicas con rutas específicas. 


### Rutas con parámetros 

Hasta ahora usé rutas fijas para navegar. Lo más normal es que estas **plantillas contengan segmentos variables** llamados parámetros. Para incorporar parámetros a tu esquema de rutas tienes que actuar en tres fases:

#### 1- Definir la parte paramétrica de la plantilla generadora de rutas

El *path* del objeto ruta se convierte en una plantilla que admite distintos valores. Así queda ahora el módulo enrutador de movimientos:

```javascript
const routes: Routes = [
  {
    path: 'movimientos',
    component: MovimientosComponent,
    children: [ // rutas hijas, se verán dentro del componente padre
      {
        path: 'nuevo', // la ruta real es movimientos/nuevo
        component: NuevoComponent
      },
      {
        path: 'lista',
        component: ListaComponent
      }
    ]
  },
  {
    path: 'movimientos/:id', // parámetro variable id    
    component: EditorComponent
  }
];
```

Los parámetros se prefijan con `:` y en cada ruta se pueden usar tantos como sea necesario. 

>Obsérvese que en este caso la ruta 'movimientos/:id' es hermana, no hija, de la primera. Por tanto se mostrará en el `router-outlet` del componente raíz. Se ha hecho así para mostrar distintas maneras de trabajar en el 'router'.

 

#### 2- Montar los enlaces asignando valores a los parámetros

Esto se puede hacer en las plantillas HTML o en por código. Siempre usando el **array de `routerLink`** sin necesidad de concatenar cadenas para para montar rutas. Este array en su segunda posición llevará un objeto que represente los valores de los parámetros. 

En un caso de navegación por código sencillo tendrá esta pinta:
```javascript
  // para ir a la ruta /movimientos/42
  this.router.navigate(['movimientos', 42])
```

Usando desde la vista en igual de sencillo con la directiva `routerLink`, como en este ejemplo HTML:

```html
<a [routerLink]="['/movimientos', 42 ]">42</a>
```

#### 3- Recuperar los valores de los parámetros a partir de las rutas
La novedad más llamativa es la presencia de `OnInit`. Es un *hook*, o evento de la vida de un componente. Este evento se ejecuta al iniciarse el componente pero cuando ya la ruta se ha resuelto completamente. 
En ese momento puedes usar `ActivatedRoute`, un servicio que entre otras cosas te dará acceso a un *observable* que emite los valores actuales de los parámetros.
Por ejemplo, esto sería el código del componente `editor.component.ts` que se activa con rutas como *'/movimientos/42'*

```javascript
import { Component, OnInit } from '@angular/core';
/** Servicio para acceder a la ruta activa */
import { ActivatedRoute } from '@angular/router';
import { DatosService } from './../datos.service';
@Component({
  selector: 'app-editor',
  templateUrl: './editor.component.html',
  styleUrls: ['./editor.component.css']
})
export class EditorComponent implements OnInit {
  public movimiento;
  constructor(
      private route: ActivatedRoute, 
      private datosService: DatosService) { 
      // constructor vacío. sólo se usa para reclamar dependencias         
  }
  ngOnInit() {
    // subscripción al observable params
    this.route.params
      .subscribe(params => {
        const _id = params['id'].toString();
        this.movimiento = this.datosService.getMovimientoBy_Id(_id);
      });
  }
}
```
> Hay una estrategia opcional que emplea `Observables` para mantener potenciales cambios de estado en los valores de los parámetros. Si te interesa busca información sobre `Router` .

### Carga diferida

Esta es la funcionalidad más esperada y que sigue siendo un *work in progress*. Cuando las aplicaciones AngularJS crecen en funcionalidad producen un **impacto negativo en la primera vista** de un usuario. Eso es debido a que *Angular* necesita disponer de todo tu código para montar el armazón de dependencias. Esto es así aunque el usuario no vaya a navegar mas que por un conjunto reducido de rutas.

En aplicaciones de *intranet* o de uso muy intensivo esto no suele ser un gran problema. Esa espera inicial de unos pocos segundos se recupera durante el uso continuado de la aplicación. Pero ciertos desarrollos realmente grandes o, sobre todo, **aplicaciones web públicas para usuarios ocasionales** necesitaban un tratamiento especial.

La solución es implementar un modelo de *lazy loading* o carga diferida. En este caso el navegador descarga el HTML y el código de la aplicación según el usuario navegue. 

En *Angular2* se incluye esta funcionalidad, pero a día de hoy aún no está disponible con `angular-cli`. Se puede *tunear WebPack* a mano para conseguirlo, pero, si no hay urgencia en salir a producción, no te lo recomiendo. Es preferible esperar un poco y usar el la solución definitiva que implemente la herramienta.

En cuanto sea *usable* actualizaré este artículo y difundiré la buena nueva a los cuatro vientos. Como sabes aún estamos esperando la primera *Release Candidate* de `angular-cli` y habrá mejoras importantes. 

Mantente a la última recibiendo el [boletín de noticias de Academia Binaria](http://academia-binaria.us4.list-manage.com/subscribe?u=c8ad2d2e7d02c26e32ce4cded&id=b67e4d2339) o siguiéndome en las [redes sociales](https://twitter.com/albertobasalo). 
