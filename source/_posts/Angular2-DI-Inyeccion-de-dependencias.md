---
title: 'Angular2 DI: Inyección de dependencias'
tags:  
- Angular2
- DI
- Tutorial
categories:
- Introducción 
permalink: angular2-di-inyeccion-de-dependencias
id: 14
updated: 2016/11/10 14:52:13
date: 2016/05/30 11:09:09
---

> ACTUALIZACIÓN: para una versión más reciente del contenido visita la página [Servicios inyectables en Angular](../servicios-inyectables-en-Angular/)

---

>Código asociado en [angular2/4-injection/](https://github.com/AcademiaBinaria/angular2/tree/master/4-injection)

AngularJS2 tiene vocación de *framework* para grandes aplicaciones de negocio. Los grandes desarrollos requieren **modularidad en el código**. En AngularJS se resuelve mediante la Inyección de Dependencias, siguiendo el conocido patrón *[Dependency Injection](https://es.wikipedia.org/wiki/Inyecci%C3%B3n_de_dependencias)*. 

Los cambios en la versión 2 son sobre todo sintácticos, pero también conceptuales. Se mantiene el mismo objetivo, permitir hacer **aplicaciones de negocio con HTML y JavaScript** de manera controlable.

<!-- more -->

## Módulos, componentes y servicios
En AngularJS 1 teníamos muy presente el concepto de **módulo**. Rara era la aplicación que no comenzase con el mítico `angular.module('miAplicacion')`. Dada la carencia de un sistema modular nativo en *JavaScript*, AngularJS nos proveía de uno propio. 

Hoy en día se nos sugiere que usemos las versiones avanzadas de *JS*, o mejor aún *TypeScript*. Estos lenguajes nos ofrecen las instrucciones `export` e `import` que permiten definir **módulos estancos en ficheros** independientes. Nunca más el engorroso envolvimiento *IIFE* `(function () { … }())`.

A partir de ahí es el programador el que selectivamente **exporta funcionalidad que importará para ser consumida** mas tarde. De este modo, Angular se desentiende de la creación de módulos de código y se centra en la comunicación entre los objetos que componen la aplicación.


### Componentes
Son los bloques de construcción agrupables en compuestos que forman la interfaz visual de una aplicación. Encapsulan **la vista, los datos y la lógica** para interactuar con el usuario. 

Ni la vista ni la lógica deben crecer y realizar tareas que no sean de su exclusiva capa de responsabilidad. Esas tareas debe ser delegadas en otros objetos. 

> Modelo de composición de componentes visuales para crear vistas complejas. La comunicación o [Data Flow](../databinding-el-flujo-de-datos-de-angular2/) se realiza de manera declarativa. Enviando datos a las propiedades de los hijos y esperando eventos con respuestas.

![Composición mediante componentes](/images/ng2-Component-DataFlow.jpg)

### Servicios
Los servicios serán **objetos especializados y reutilizables** por otros servicios y componentes. En su definición debes aplicar los mismos principios de arquitectura de software que ya conoces y aplicas en *lenguajes clásicos tipo Java o C#.net*.

La sintaxis de la comunicación involucra varios bloques en ambos lados del canal: 

- la definición del servicio decorado como `@Injectable()` en el proveedor, 
- la exportación en el proveedor y la importación en el consumidor, 
- el registro en el array `providers:[]` del componente consumidor (o mejor del módulo) conumidor y 
- el consumo en el constructor del componente o servicio consumidor.

```javascript
import { Component, OnInit } from '@angular/core';
import { MovimientosService, Movimiento } from '../shared/';
@Component({
  selector: 'movimiento',
  templateUrl: 'movimiento.component.html',
  providers: [MovimientosService] 
})
export class MovimientoComponent {
  movimiento: Movimiento
  constructor(public movimientosService: MovimientosService) { }
  guardarMovimiento() {
    this.movimientosService.guardarMovimiento(this.movimiento)
  }
}
```

![Inyección de servicios en componentes ](/images/ng2-DI-component-service.jpg)

### Directivas
Las directivas eran el ADN de AngularJS1. Ahora han mutado en **componentes reutilizables** como elementos en las vistas de otros componentes de rango superior. Pero la idea es la misma, desglosar las plantillas de HTML en bloques con un propósito único. 

La sintaxis de las directivas es similar: 

- la definición del servicio decorado como `@Injectable()` en el componente hijo, 
- la exportación del hijo y la importación en el padre, 
- el registro en el array `directives:[]` del componente padre (o del módulo padre) y 
- el consumo declarativo en la plantilla de la vista padre.

```javascript
import { Component } from '@angular/core';
import { MovimientoComponent } from './movimiento';
@Component({
  selector: 'injection-app',
  template: '<h1>
              {{titulo}}
            </h1>
            <movimiento></movimiento>',
  directives:[MovimientoComponent]
})
export class InjectionAppComponent {
  titulo = 'Inyectores listos!';
}
```
![Inyección de directivas en componentes](/images/ng2-DI-component-directive.jpg)

## Registro

La inyección de las dependencias **funciona de manera jerárquica** en AngularJS 2. Cuando un módulo registra una dependencia la pone a disposición de todos sus componentes hijos. Es más, si algún hijo la volviese a registrar se le proveería de otra instancia. 

Esta copia puede provocar efectos colaterales indeseados. Para **compartir datos** o ahorrar memoria se recomienda registrar las dependencias lo más arriba posible. Esto siempre sin sacrificar la modularidad o la escalabilidad de aplicaciones que requieran *lazy loading*.

> Atención a la copia de *routeService* que se registra por segunda vez. No importa que ya la haya registrado su padre. Otro registro implica otra instancia.

![Jerarquía de dependencias en AngularJS](/images/ng2-Arbol-de-dependencias.jpg)

Para ciertos casos, AngularJS 2 permite modelos de inyección más avanzados. Mediante el uso de **factorías y el registro de cadenas** con los nombres de los servicios facilita la inyección a voluntad o la carga diferida.

>Algunas de estas posibilidades están siendo retocadas durante la actual *Release Candidate*, y se esperan cambios de cara a las próxima versión estable.

El viejo principio de *divide y vencerás* se aplica rotundamente en las aplicaciones Angular 2. Si creas **módulos reutilizables** estarás sentando las bases para crear grandes aplicaciones de negocio con AngularJS.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>

---

### ACTUALIZACIÓN

Para una versión más reciente del contenido visita la página [Servicios inyectables en Angular](../servicios-inyectables-en-Angular/)