---
title: DataBinding el flujo de datos de Angular2
tags:  
- Angular2
categories:
- Introducción 
permalink: databinding-el-flujo-de-datos-de-angular2
id: 13
updated: 2016/11/10 14:56:46
date: 2016/05/25 17:26:32
---

> Código asociado en [angular2/3-databinding/](https://github.com/AcademiaBinaria/angular2/tree/master/3-databinding) 

El *doble binding* o enlace vista controlador en ambos sentidos es una de las claves del éxito de AngularJS. Usando la célebre directiva `ng-model` enganchamos **una propiedad del modelo con un control de la vista**. De manera *automágica* Angular se suscribe a cambios en el DOM y observa el estado del modelo manteniéndolos en sincronía. 

Pero esto tiene un coste en rendimiento que Angular2 supera con un patrón distinto para el control del **flujo de datos** entre la vista y el modelo. Este nuevo paradigma ofrece soluciones para distintos escenarios:

![Flujo de datos](/images/ng2-Flujo-de-datos.jpg)

<!-- more -->

## Sólo lectura: modelo hacia la vista
### 1- Interpolación
En este primer caso todo nos resultará muy familiar. En efecto la sintáxis de interpolación es la mismoa que en AngularJS 1.x. eso si, en este caso y por defecto, los datos son de sólo lectura y Angular no se preocupa de actualizar la variable.
```html
     <p>Hola {{nombreDelProgramador}} bienvenido a Angular2</p>
     <p>Fecha de nacimiento: {{fechaDeNacimiento | date}}</p>
```
### 2- Enlace a propiedades
Es la comunicación básica hacia la vista, hacia el usuario. En este caso cualquier atributo de un elemento HTML puede enlazarse al valor de una propiedad encerrándola entre corchetes y asignándole una expresión. `[propiedad]="expresion"`
```html
     <p>Hola <input [value]="nombreDelProgramador" readonly ></input> bienvenido a Angular2</p>
     <a [href]="url-academia-binaria">Academia Binaria</a>
     <div [hidden]="usuarioAutenticado">Identifícate</div>     
     <div [hidden]="!usuarioAutenticado">Hola {{nombreUsuario}}</div>
```

## Sólo escritura: de la vista hacia el modelo
### 1- Eventos
La comunicación desde la vista hacia el modelo se realiza mediante eventos. Es una buena práctica llamar a funciones del componente de forma declarativa en la vista. 
La sintaxis requiere que se nombre el evento entre paréntesis y se le asigne una expresión como valor. `(evento)="expresion"`
```html
     <input (keyup)="onKey($event)" />
     <input #nombre
      (keyup.enter)="propiedad=nombre.value"
      (blur)="propiedad=nombre.value">
     <button (click)=lanzarCohete()>Lanzar cohete</button>
```
## Lectura y escritura: bidireccional
### 1- Enlace doble.
Este es el caso más común en la edición de formularios. Y es el equivalente al viejo y glorioso *doble binding*. Es la combinación de los dos anteriores y eso se refleja en la sintaxis. Recordad `[]` para leer propiedades y `()` para enviar datos en respuesta a eventos: el resultado es la llamada *banana in a box* `[()]`. En este caso se completa con la directiva ngModel y la propiedad enlazada. `[(ngModel)]="propiedad"`
```html
  <input type="text"  [(ngModel)]="nombreDelProgramador" >
  Hola {{nombreDelProgramador}}
```

## Resumen
Este es un ejemplo recopilatorio de las capacidades de Angular2 en cuanto a la sintaxis declarativa en las plantillas HTML de los componentes. Recordad este *mantra* que revisaremos durante la composición de componentes en aplicaciones complejas:
> Los datos fluyen hacia las propiedades de los componentes hijos. Los eventos brotan desde los hijos hacia los padres. Sólo usamos enlace doble en los formularios que así lo requieran.

```html
<h1>
  <!--Interpolación de variables definidas en el modelo del componente-->
  {{title}}
</h1>
<form>
  <label>¿Cómo te llamas?</label>
  <!--Enlace doble (lectura y escritura) entre la vista y el modelo-->
  <input type="text" [(ngModel)]="aprendiz" />
  <p>Bienvenido a Angular 2 {{ aprendiz }} </p>
  
  <!--Expresiones-->
  <p>Soy capaz de multiplicar por {{1 * 2}} tus habilidades </p>
  <!--Eventos-->
  <button (click)="visible=true">Saludar</button>
  <!--Propiedades-->
  <p [hidden]="!visible">Hola Mundo!!!</p>
</form>
```

En estos ejemplos se han visto propiedades y eventos estándar. Pero todo lo dicho es aplicable a las **propiedades y eventos** especialmente creados para tus componentes.

El objetivo, conseguido, es aumentar el **rendimiento**. Y un efecto colateral es la **simplificación del API** de AngularJS. Desaparecen casi todas las directivas estructurales. Especialmente las famosas `ng-click ng-blur` y demás directivas asociadas a eventos. Tampoco se necesitan más las directivas de `ng-show` y `ng-hide`.

Las únicas directivas estructurales que permanecen son `*ngIf` `*ngSwitch` y `*ngFor`. Pero esas merecen tratamiento aparte.
