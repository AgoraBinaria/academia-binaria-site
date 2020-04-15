---
title: Formularios, tablas y modelos de datos en Angular
permalink: formularios-tablas-y-modelos-de-datos-en-angular
date: 2020-04-15 15:17:37
tags:
  - Angular
  - Forms
  - Tutorial
  - Introducción
  - Angular9
  - Angular2
categories:
  - [Tutorial, Angular]
thumbnail: /css/images/angular-3_data.png
---

![formularios-tablas-y-modelos-de-datos-en-angular](/images/tutorial-angular-3_data.png)

Las **aplicaciones Angular 9 son excelentes para el tratamiento de datos** en el navegador. Su razón de ser fue la recogida de información mediante formularios y la presentación de páginas dinámicas de forma sencilla.

Vamos a ver cómo la librería `@angular/forms` enlaza **las vistas, los controladores y los modelos**; y cómo se hace la presentación de datos en **listas y tablas**.

<!-- more -->

Partiendo de la aplicación tal como quedó en [Páginas y rutas Angular SPA](../paginas-y-rutas-angular-spa/), al finalizar tendrás una aplicación que recoge y presenta datos.

> Código asociado a este tutorial en _GitHub_: [AcademiaBinaria/angular-basic](https://github.com/AcademiaBinaria/angular-basic/)

# 1. Binding

## 1.0 Base

**Los formularios son el punto de entrada** de información a nuestros sistemas. Llevan con nosotros desde el inicio de la propia informática y se han comido una buena parte del tiempo de programación. En _Angular_ han prestado una especial atención a ellos facilitando su desarrollo, **desde pantallas simples hasta complejos procesos**.

Vamos con simple ejemplo para recopilar información de contactos personales. Creamos una nueva ruta funcional para la gestión de contactos. Requiere ruta, enlace, módulo y componente.

```bash
ng g m contacts --route contacts --module app-routing.module
```

Como ya sabemos, además de la creación del módulo y el componente, esto altera los ficheros de enrutado. Así tenemos lo siguiente en `app-routing` y en `contacts-routing`:

```typescript
  // app-routing
  {
    path: 'contacts',
    loadChildren: () => import('./contacts/contacts.module').then(m => m.ContactsModule)
  },
  // contacts-routing
  {
    path: '',
    component: ContactsComponent
  }
```
Hay que añadir una entrada en el `SideComponent` y ya tenemos listo el armazón para continuar.

```html
<li><a routerLink="contacts">Forms</a></li>
```

## 1.1 Directivas

Antes de pedir información, vamos a prepara el terreno. Necesitamos algunos textos para instruir al usuario y, sobre todo, un lugar en dónde recoger lo que nos escriba. Para ello atacaremos a la clase controladora del nuevo componente.

> Para empezar agregamos algunas propiedades. En `contacts.component.ts`:

```typescript
header = 'Contacts';
description = 'Manage your contact list';
numberOfContacts = 0;
counterStyleColor = 'green';
counterClass = 'warning';
formHidden = false;
```

### 1.1.1 Enlace del modelo hacia la vista

Estas propiedades públicas son visibles desde la vista HTML. Así, en `contacts.component.html` mostramos cabeceras con estilo como este.

```html
<h3>
  {{ header }}: {{ description | uppercase }}
</h3>
<p [style.color]="counterStyleColor">You have <mark
        class="{{ counterClass }}">{{ numberOfContacts }}</mark>
  contacts right now.</p>
```

Este código HTML es básicamente estándar, pero está reforzado con algunos símbolos que pueden resultar extraños. Son **las directivas de Angular**. Se trata de atributos fuente que una vez compilados generan funcionalidad extra.

La manera más directa de hacerlo es representar en la vista algún dato del modelo. Y eso se hace con las expresiones de interpolación entre llaves. `{{ expression }}` Suelen usarse para aportar contenido a elementos html, aunque también a cualquier atributo.

A la hora de pintar datos, a veces querremos transformarlos antes. Para eso emplearemos **los pipes de Angular**. Son funciones que adjuntaremos a las expresiones usando el carácter especial `|` llamado _pipe_. La entrada de la función es lo que hay a la izquierda de la tubería. La salida de la función es lo que se muestra.

En Angular disponemos de suficientes pipes estándar para cubrir los casos básicos de presentación de textos, fechas, monedas y demás.

También tenemos formas de asignar contenido dinámico a los atributos. Basta con envolvernos entre corchetes y su valor se calcula como una expresión. Como norma general sería algo así: `[atribute]="expresion"`

### 1.1.2 Enlace de la vista hacia el modelo

Toca ahora darle la opción al usuario de actuar sobre los datos desde la vista. Por ejemplo en `contacts.component.html` hemos creado un par de botones cuya intención es mostrar u ocultar un formulario sobre el que trabajaremos más adelante.

```html
<input
  value="Show Form"
  class="primary"
  type="button"
  (click)="formHidden=false"
/>
<input
  value="Hide Form"
  class="inverse"
  type="button"
  (click)="formHidden=true"
/>
<form [ngClass]="{'hidden':formHidden}">
  <fieldset>
    <legend>Contact Form</legend>
  </fieldset>
</form>
```

Para ello usaremos los eventos estándar del HTML, como el _click_ de los _input_. Pero, de nuevo, usaremos símbolos propios de Angular como los paréntesis `(eventName)`. En esta ocasión lo que asignaremos será un instrucción, que será ejecutada cuando el evento se dispare.

Para ver el efecto completo, echamos mano de otra directiva propia de Angular: la `ngClass`. Esta nos sirve para aplicar estilos CSS conducidos por los datos. Se aplican o no en función de que se cumplan ciertas condiciones dinámicamente. En este caso el estilo _hidden_.

# 2. Doble Binding

Hemos visto lo fácil que es mostrar datos en una pantalla. Y tampoco resulta complejo responder a eventos del usuario. Digamos que tenemos mecanismos para realizar el enlace, _binding_ en el argot de Angular, entre la vista y el modelo.

La idea es juntar ambos mecanismos en algo que permita enlazar vista y modelo en ambos sentidos. De esta forma podremos mantener sincronizados lo que el usuario ve y lo que realmente se está procesando.

## 2.1 NgModel

La directiva _ngModel_ viene en un módulo del framework llamado `FormsModule`. Hay que importarlo para poder usar su contenido, tal como hemos hecho en `contacts.module.ts`

```typescript
import { CommonModule } from '@angular/common';
import { NgModule } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { ContactsRoutingModule } from './contacts-routing.module';

@NgModule({
  imports: [
    CommonModule,
    ContactsRoutingModule,
    FormsModule
  ]
})
export class ContactsModule { }
```

A partir de ese momento podemos invocar sus directivas, yb vamos a empezar por la más utilizada y famosa: `ngModel`.


### Banana in a box [()]

Pero ojo, esta potente directiva no se jea utilizar así como así. Hay que enjaularla convenientemente entre una doble barrera de corchetes y paréntesis `[()]`. Para recordarlo se puso de moda la frase _Banana in a box_ que hace referencia al _paréntesis dentro del corchete_

```html
<input [(ngModel)]="model.property"/>
```

Usa la comunicación en ambos sentidos

- **(banana)** : de la vista al modelo
- **[box]** : del modelo a la vista

#### Modelo

> La directiva se asocia con una propiedad del controlador...
> o mejor aún, con una **propiedad del modelo** del controlador

Por ejemplo podemos crear un objeto literal para representar a un contacto y trabajar sobre él

```typescript
public contact = { name: '' };
```

#### Directiva

Y enlazarlo con la vista para que siempre estén sincronizados.

```html
<section>
  <label for="name">Name</label>
  <input
    name="name"
    type="text"
    [(ngModel)]="contact.name"
    placeholder="Contact name"
  />
</section>
```

#### Espía

Mientras desarrollas, es frecuente que quieras visualizar el valor de cualquier propiedad en tiempo de ejecución. Algo así como un `console.log()` en la pantalla.

```html
<pre>{{ contact | json }}</pre>
```

El anterior elemento se un espía perfecto de la actividad del usuario sobre el formulario. Una cosa más, acuérdate de quitarlo antes de enviar a producción.

## 2.2 Form

La técnica básica de enlazar _inputs_ con propiedades puedes extenderla cuanto quieras. Pero pronto echarás en falta los _check boxes, radio buttons_ y demás. Hay más usos de las directivas en los formularios.

Vamos a ver cómo usarlos para actuar sobre el siguiente modelo:

```typescript
public contact = { name: '', isVIP: false, gender: '' };
```

### 2.2.1 CheckBox

Quizá nos venga bien un _checkbox_ para saber si es o no un VIP.

```html
<section>
  <label for="isVIP">Is V.I.P.</label>
  <input name="isVIP" type="checkbox" [(ngModel)]="contact.isVIP" />
</section>
```

### 2.2.2 Radio Buttons

Y un par de _radio buttons_ para el género.

```html
<section>
  <label for="gender">Gender</label>
  <input name="gender" value="m" type="radio" [(ngModel)]="contact.gender" />
  <i>Male</i>
  <input name="gender" value="f" type="radio" [(ngModel)]="contact.gender" />
  <i>Female</i>
</section>
```

# 3. Estructuras

Mantén siempre en mete que el HTML que escribes es código fuente. No es, aún, el HTML que renderizará el navegador. Es casi casi un lenguaje de programación per se. Con sus expresiones, instrucciones y estructuras.

## 3.1 \*ngFor

Vamos a empezar por las **estructuras repetitivas** presentando la Directiva estructural `*ngFor`.

Avanzamos con el modelo al que agregamos posibles estados laborales de nuestros contactos:

```typescript
public workStatuses = [
  { id: 0, description: 'unknow' },
  { id: 1, description: 'student' },
  { id: 2, description: 'unemployed' },
  { id: 3, description: 'employed' }
];
public contact = { name: '', isVIP: false, gender: '', workStatus: 0 };
```

> \*ngFor

```html
<section>
  <label for="workStatus">Work Status</label>
  <select name="workStatus" [(ngModel)]="contact.workStatus">
    <option *ngFor="let status of workStatuses" [value]="status.id">
      <span>{{ status.description }}</span>
    </option>
  </select>
</section>
```

Es un bucle de toda la vida vamos. Pero en em medio del HTML. Lo que hace es dar vueltas sobre un array y agregar un nodo HTML para cada elemento del array.

> let **iterador** of **iterable**

## 3.2 \*ngIf

Y ahora le toca a las **estructuras condicionales** y para ello tenemos la directiva estructural `*ngIf`

Vamos un paso más en el modelo y supongamos que queremos preguntar por la empresa actual si nuestro contacto está trabajando, y en caso contrario preguntarle por sus estudios.

```typescript
public contact = {
  name: '',
  isVIP: false,
  gender: '',
  workStatus: '0',
  company: '',
  education: ''
};
```

---

> \*ngIf

```html
<section *ngIf="contact.workStatus=='3'; else education">
  <label for="company">Company Name</label>
  <input name="company" type="text" [(ngModel)]="contact.company" />
</section>
<ng-template #education>
  <section>
    <label for="education">Education</label>
    <input name="education"
            type="text"
            [(ngModel)]="contact.education"
            placeholder="Education" />
  </section>
</ng-template>
```

Vemos incrustado en HTML el típico _if else_ de la programación estructurada. Quizá llame la atención la manera de tratar el _else_. Por ahora digamos que aquellos nodos del árbol que no tienen garantizada su existencia deben recogerse dentro de un elemento propio del Angular, las `ng-template`.

Otra directiva símbolo que merece la pena mencionar es el `#`. Actúa como un identificador que una vez aplicado a un elemento se puede usar para acceder a el desde cualquier parte del componente.

En resumen:

> if **condition** else **template**

> > también hay _\*ngSwitch_

---

# 4. Modelo y controlador

Este punto nu es de Angular propiamente, pero sí del lenguaje que han escogido para que desarrollemos nuestras apps: el **TypeScript**.

Es como un cruce entre _JavaScript_ y _Java_ o _C#_. Digamos que le aporta tipos estáticos y mejoras en cuanto expresividad si queremos usar clases o una programación más orientada a objetos. Es un lenguaje de ayuda al programador. Realmente puedes usar sintaxis pura de JavaScript, porque el typeScript es JavaScript con anotaciones de opcionales de tipos.

Angular viene ya con las herramientas y configuraciones necesarias para _transpilar_ el TypeScript a JavaScript.

## 4.1 Interfaces y modelos

> Mejor interface que clase

Se aconseja por una buena razón. Las interfaces en JavaScript no existen. Así, mientras desarrollamos nos ayuda con _intellisense_ y noes protegen de asignaciones indebidas. Pero, en ejecución no pesan porque no tienen contrapartida.

```typescript
export interface Option {
  id: number;
  description: string;
}

export interface Contact {
  name: string;
  isVIP: boolean;
  gender: string;
  workStatus: number | string;
  company: string;
  education: string;
}
```

> tipos compuestos `number | string`

Este es un lenguaje que realmente sólo aplica mientras desarrollas. Así que permite hacer diabluras con los tipos.

Se usan para tipificar las propiedades

```typescript
public workStatuses: Option[] = [
    { id: 0, description: 'unknow' },
    { id: 1, description: 'student' },
    { id: 2, description: 'unemployed' },
    { id: 3, description: 'employed' }
  ];
public contact: Contact = {
    name: '',
    isVIP: false,
    gender: '',
    workStatus: 0,
    company: '',
    education: ''
  };
public contacts: Contact[] = [];
```

## 4.2 ViewModel en el controlador

El controlador del componente es una clase. Por tanto tiene no solo propiedades, sino también métodos.

```typescript
saveContact() {
  this.contacts.push({ ...this.contact });
  this.updateCounter();
}

private updateCounter() {
  this.numberOfContacts = this.contacts.length;
  this.counterClass = this.numberOfContacts === 0 ? 'warning' : 'success';
}
```

Los métodos públicos pueden, y deben, ser invocados desde las vistas. De forma que las expresiones asignadas a los eventos sean simples llamadas a métodos para que hagan el trabajo sucio.

```html
<input value="Save" type="submit" (click)="saveContact()" />
```

### OnInit

Ya hemos visto que en TypeScript, las clases puede implementar interfaces. Angular nos facilita unos cuantos para usar como _hooks_ en determinados momentos dl ciclo de vida de una componente.

En particular hay uno que ya vienen pre implementado por el generador del CLI. Se llama `OnInit` y obliga a disponer de un método público llamado `ngOnInit()`. Lo que programes dentro será ejecutado durante la inicialización del componente.

Es una práctica recomendable usar ese método en lugar del constructor para desplegar nuestra lógica de inicio. La razón es que el constructor se ejecuta antes de la existencia completa de la vista y eso puede generar inconsistencias.

```typescript
public workStatuses: Option[];
public contact: Contact;
public contacts: Contact[];
constructor() {}
public ngOnInit() {
  this.workStatuses = [
    { id: 0, description: 'unknow' },
    { id: 1, description: 'student' },
    { id: 2, description: 'unemployed' },
    { id: 3, description: 'employed' }
  ];
  this.contact = {
    name: '',
    isVIP: false,
    gender: '',
    workStatus: 0,
    company: '',
    education: ''
  };
  this.contacts = [];
}
```

### Repasamos

Agregamos una funcionalidad que nos obliga a repasar todo lo aprendido. Un listado en el que mostrar los contactos que se recogen del formulario.

```html
<ul *ngIf="contacts.length>0; else empty">
  <li *ngFor="let contact of contacts">
    <span>{{ contact.name }}</span>
    <input value="Delete" type="button" (click)="deleteContact(contact)" />
  </li>
</ul>
<ng-template #empty> <i>No contacts yet</i> </ng-template>
```

```typescript
deleteContact(contact: Contact) {
  this.contacts = this.contacts.filter(c => c.name !== contact.name);
  this.updateCounter();
}
```

Mira el código completo de **la clase** `ContactsComponent`en el fichero `contacts.component.ts` para tener una visión completa del componente. Como ves, **las propiedades** `header, numberOfContacts, formHidden, contacts ...` se corresponden con las utilizadas en las directivas de enlace en la vista. Mientras que **los métodos** `saveContact(), deleteContact()` son invocados desde eventos de elementos del _html_.

Juntos, **la vista y su clase controladora**, resuelven un problema de interacción con el usuario **creando un componente**. Todas las páginas que diseñes serán variaciones y composiciones de estos componentes.

> Y esto es sólo el comienzo. La idea de componente será fundamental en la web del mañana para la creación de páginas mediante `web components`. Pero eso ya se verá más adelante...

Ahora tienes una aplicación en _Angular 9_ que recoge y muestra datos. Sigue esta serie para añadirle [Flujo de datos entre componentes Angular](../flujo-de-datos-entre-componentes-angular/) mientras aprendes a programar con Angular8. Todos esos detalles se tratan en el [curso básico online](https://www.trainingit.es/curso-angular-basico/?promo=angular.builders) que imparto con TrainingIT o a medida para tu empresa.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>
