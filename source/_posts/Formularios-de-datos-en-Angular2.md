---
title: Formularios de datos en Angular2
tags:  
- Angular2
categories:
- Introducción 
permalink: formularios-de-datos-en-angular2
id: 21
updated: 2016-06-22 18:05:58
date: 2016-06-22 16:32:34
thumbnail: /css/images/angular.jpg
---

Los formularios en AngularJS fueron un éxito inicial debido entre otras coas al *double binding*. Otra cosa que no fue nunca simple es la **validación de controles en los formularios**. En Angular 2 y con la última versión RC3 la cosa mejora, pero es aún un *Work in progress.*
  
<!-- more -->

### Dependencias
Con el lío de versiones de los últimos días, conviene que os muestre con qué he hecho mi prueba.
```json
"dependencies": {
    "@angular/common": "2.0.0-rc.3",
    "@angular/compiler": "2.0.0-rc.3",
    "@angular/core": "2.0.0-rc.3",
    "@angular/forms": "^0.1.1",
    "@angular/http": "2.0.0-rc.3",
    "@angular/platform-browser": "2.0.0-rc.3",
    "@angular/platform-browser-dynamic": "2.0.0-rc.3",
    "@angular/router": "3.0.0-alpha.7",
    "es6-shim": "^0.35.0",
    "reflect-metadata": "0.1.3",
    "rxjs": "5.0.0-beta.6",
    "systemjs": "0.19.26",
    "zone.js": "^0.6.12"
  }
```

Como veis están actualizadas a la `RC.3` y `forms 0.1.1`.
> Si usas **SystemJS**, acuérdate de incluir `@angular/forms` en el fichero `system-config.json`

En el `main.ts` debemos registrar los *providers* que permiten la convivencia de la actual versión de *forms* y la anterior (de la semana pasada) ya obsoleta.

```javascript
import { disableDeprecatedForms, provideForms } from '@angular/forms';
bootstrap(CashFlowAppComponent,[
  disableDeprecatedForms(),
  provideForms()
]);
```

### Plantillas html para hacer formularios
Vamos a tratar de mantener el **html limpio** hasta dónde sea posible. En un futuro espero que incluso sea prescindible y que se pueda generar de forma automática a partir de datos de configuración.
```html
<form [formGroup]="formularioMovimiento">
  <label>Tipo:</label>
  <input type="text" formControlName="tipo">
  <p>
    <span *ngIf="formularioMovimiento.controls['tipo'].touched && !formularioMovimiento.controls['tipo'].valid">
      Necesitamos el tipo de movimiento
    </span>
  </p>
  <label>Categoría:</label>
  <input type="text" formControlName="categoria"><p></p>
  <label>Fecha:</label>
  <input type="date" formControlName="fecha"><p></p>
  <label>Importe:</label>
  <input type="number" formControlName="importe"> <p>
    <span *ngIf="formularioMovimiento.controls['importe'].touched && !formularioMovimiento.controls['importe'].valid">
      Necesitamos el tipo de movimiento
    </span>
  </p>
  <button 
    type="submit" 
    [disabled]="!formularioMovimiento.valid"
    (click)="guardarMovimiento()" >Guardar
  </button>
</form>
```

Se incluyen la lógica para mostrar mensajes de validación y activación del botón de guardado. He optado por no usar la definción de variables en la plantilla mediante `#`.

### Lógica y datos en el componente
Trataremos de llevar la **lógica de generación y validación de datos** al componente y programarlo en TypeScript. Puede ser por mi pasado *backender* pero me encuentro más cómodo cuanto más lejos de un lenguaje de marcas.
 
Para empezar necesitamos registrar las herramientas. El proveedor `FormBuilder` nos ayuda a definir los controles asociados al formulario y sus validaciones.

```javascript
import { REACTIVE_FORM_DIRECTIVES, FormBuilder, FormGroup, Validators  } from '@angular/forms';
@Component({
  directives: [REACTIVE_FORM_DIRECTIVES],
  providers:[FormBuilder]
})
```
En el evento `OnInit` construimos la estructura de soporte al formulario. Se puede hacer mucho más compleja, mediante la inclusión de grupos de controles anidados.

```javascript
ngOnInit() {
    this.formularioMovimiento = this.formBuilder.group({
      tipo: ['',Validators.required],
      categoria: [],
      fecha: [],
      importe:['',Validators.required]
    });
  }
```

Antes de enviar datos a un serviciop se pueden hacer validaciones y ttransformacipnes previas. El acceso a los valores del formulario se hace través de su propiedad `.value`. Con esto ya realizaremos los envíos a los servicios...

```javascript
guardarMovimiento() {       
 this.miServicio.Guardar(this.formularioMovimiento.value);
}
```
### Validación
La validación temprana es el objetivo fundamental de un desarrollo de formularios. Es buena para el usuario y buena para el sistema. Angular2 facilita la implementación de las validaciones estándar y permite la creación de las tuyas propias.

Lo veremos en próximas entregas. Sígueme en [mi cuenta de twitter](https://twitter.com/albertobasalo) o suscríbete a la [revista mensual de Academia Binaria](http://academia-binaria.us4.list-manage.com/subscribe?u=c8ad2d2e7d02c26e32ce4cded&amp;id=b67e4d2339) y serás el primero en ser informado.

*Keep coding, keep learning.*
