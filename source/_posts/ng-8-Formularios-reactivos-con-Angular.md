---
title: Formularios reactivos con Angular
permalink: formularios-reactivos-con-Angular
date: 2018-05-03 10:59:27
tags:  
- Angular
- Angular6
- Angular5
- Angular2
- reactiveForms
- Tutorial
- Introducción
categories:
- [Tutorial, Angular] 
thumbnail: /css/images/angular_8_reactive.png
---

![Tutorial Angular 8-reactive](/images/tutorial-angular_8_reactive.png)

El **doble enlace automático** entre elementos html y propiedades de objetos fue el primer gran éxito de **Angular**. Ese _doble-binding_ facilita mucho el desarrollo de formularios. Pero esa magia tienen un coste en escalabilidad; impacta en el tiempo de ejecución y además dificulta la validación y el mantenimiento de formularios complejos.

La solución pasa por desacoplar el modelo y la vista, introduciendo una capa que gestione ese doble enlace. Los servicios y directivas del módulo `ReactiveFormsModule` que viene en la librería `@angular/forms` permiten programar **formularios reactivos conducidos por el código**.

<!-- more -->

Partiendo de la aplicación tal cómo quedó en [Vigilancia y seguridad en Angular](../vigilancia-y-seguridad-en-Angular/). Al finalizar tendrás una aplicación con formularios _model driven_ fáciles de mantener y validar.

> Código asociado a este artículo en _GitHub_: [AcademiaBinaria/kakebo/8-reactive](https://github.com/AcademiaBinaria/kakebo/tree/8-reactive)


# 1 Desacople

La directiva `[(ngModel)]="model.property"` con su popular _banana in a box_ establece el doble enlace entre el elemento de la vista al que se le aplica y una propiedad del modelo. Los cambios en la vista son trasladados automáticamente al modelo, y al revés; cualquier cambio en el modelo se refleja inmediatamente en la vista.

Se puede establecer validaciones y configurar el evento que dispara las actualizaciones; todo ello usando más y más atributos y directivas en la plantilla. Son los formularios _template driven_ que degeneran en un html farragoso y difícil de mantener.

## 1.1 Form Builder

Entra en acción el  `FormBuilder`, un servicio del que han de depender los componentes que quieran desacoplar el modelo de la vista. Se usa para construir un formulario creando un `FormGroup`, o grupo de controles, que realiza un seguimiento del valor y estado de validez de los datos.

Veamos un ejemplo mínimo de su declaración. 

```typescript
import { FormBuilder, FormGroup } from '@angular/forms';
public form: FormGroup;
constructor(private formBuilder: FormBuilder) {}
public ngOnInit() {
  this.form = this.formBuilder.group({});
}
```

# 2 Form Group

El formulario se define como un grupo de controles. Cada control tendrá un nombre y una configuración. Esa definición permite establecer un valor inicial al control y asignarle validaciones.

En este paso tenemos a disposición varias sobrecargas para configurar con mayor o menor detalle el control.

## 2.1 Default data
Para empezar es fácil asignar valores por defecto. Incluso es un buen momento para modificar o transformar datos previos para ajustarlos a cómo los verá el usuario

```typescript
this.name = 'ALBERTO';
this.form = this.formBuilder.group({
  email: 'info@angular.io',
  name: this.name.toLowerCase(),
  registeredOn : new Date().toISOString().substring(0, 10)
  password: ''
});
```

## 2.1 Enlace en la vista

Mientras tanto en la vista html... Este trabajo previo y extra que tienes que hacer en el controlador se recompensa con una mayor limpieza en la vista. Lo único necesario será asignar por nombre el elemento html con el control que lo gestionará.

>Para ello usaremos dos directivas que vienen dentro del módulo _reactivo_ son `[formGroup]="objetoFormulario"` para el formulario en su conjunto, y `formControlName="nombreDelControl"` para cada control.

```html
<form [formGroup]="form">
  <label for="email">E-mail</label>
  <input name="email"
         formControlName="email"
         type="email" />
  <label for="name">Name</label>
  <input name="name"
         formControlName="name"
         type="text" />
  <label for="registeredOn">Registered On</label>
  <input name="registeredOn"
         formControlName="registeredOn"
         type="date" />
  <label for="password">Password</label>
  <input name="password"
         formControlName="password"
         type="password" />
</form>
```

# 2 Validación de formularios

La validación es una pieza clave de la entrada de datos en cualquier aplicación. Es el primer **frente de defensa ante errores de usuarios**; involuntarios o deliberados.

Dichas validaciones se solían realizar agregando atributos html tales como el conocido `required`. Pero todo eso ahora se traslada a la configuración de cada control, dónde podrás establecer un o varias reglas de validación. 

>De nuevo tienes distintas sobrecargas que te permiten resolver limpiamente casos sencillos de una sola validación, o usar baterías de reglas. Las reglas son funciones y el objeto `Validators` del _framework_ viene con las más comunes listas para usar.  

```typescript
this.form = this.formBuilder.group({
  email: [
    'info@angular.io', 
    [ Validators.required, Validators.email ]
  ],
  name: [
    this.name.toLowerCase(),
    Validators.required
  ],
  registeredOn : new Date().toISOString().substring(0, 10)
  password: [
    '', 
    [ Validators.required, Validators.minLength(4) ]
  ]
});
```
A estas validaciones integradas se puede añadir otras creadas por el programador. Incluso con ejecución asíncrona para validaciones realizadas en el servidor.

# 3 Estados

Los formularios y controles reactivos están gestionados por máquinas de estados que determinan en todo momento la situación de cada control y del formulario en si mismo.

## 3.1 Estados de validación

Al establecer una más reglas para uno o más controles activamos el sistema de chequeo y control del estado de cada control y del formulario en su conjunto.

La máquina de estados de validación contempla los siguientes mutuamente excluyentes:

- **VALID**: el control ha pasado todos los chequeos
- **INVALID**: el control ha fallado al menos en una regla.
- **PENDING**: el control está en medio de un proceso de validación
- **DISABLED**: el control está desactivado y exento de validación

Cuando un control incumple con alguna regla de validación, estas se reflejan en su propiedad `errors` que será un objeto con una propiedad por cada regla insatisfecha y un valor o mensaje de ayuda guardado en dicha propiedad.

## 3.2 Estados de modificación

Los controles, y el formulario, se someten a otra máquina que monitoriza el valor del control y sus cambios. 

La máquina de estados de cambio contempla entre otros los siguientes:

- **PRINSTINE**: el valor del control no ha sido cambiado por el usuario
- **DIRTY**: el usuario ha modificado el valor del control.
- **TOUCHED**: el usuario ha lanzado un evento `blur` sobre el control.
- **UNTOUCHED**: el usuario no ha lanzado un evento `blur` sobre el control.

Como en el caso de los estados de validación, el formulario también se somete a estos estados en función de cómo estén sus controles.

# 4 Valor

Este sistema de gestión de los controles del formulario oculta la parte más valiosa, el valor que se pretende almacenar, en una la propiedad `value` del formulario. 

Contendrá un objeto con las mismas propiedades usadas durante la definición del formulario, cada una con el valor actual del control asociado.

Un ejemplo típico sueles ser como la siguiente vista y su controlador:

```html
<form [formGroup]="form"
      (submit)="onSubmit(form.value)">
  <label for="email">E-mail</label>
  <input name="email"
         formControlName="email"
         type="email" />
 <button type="submit"
        [disabled]="form.invalid">Save</button>
</form>
```

```typescript
public onSubmit(formValue: any) {
  console.log(formValue);
  // { email:'info@angular.io' }
}
```

Ya tenemos formularios reactivos conducidos por los datos que te permitirán construir pantallas complejas manteniendo el control en el modelo y dejando la vista despejada. 

Con esto completas tu formación y dispones de conocimiento para crear aplicaciones Angular. Repasa esta serie [tutorial de introducción a Angular](../categories/Tutorial/Angular/) verás como aprendes a programar con Angular.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>
