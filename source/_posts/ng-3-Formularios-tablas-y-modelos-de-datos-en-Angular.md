---
title: Formularios, tablas y modelos de datos en Angular
permalink: formularios-tablas-y-modelos-de-datos-en-angular
date: 2020-03-10 09:17:37
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

Creamos una nueva ruta funcional para la gestión de contactos. Requiere ruta, enlace, módulo y componente.

```bash
ng g m contacts --routing true --route contacts --module app-routing.module
```

--

En `app-routing` y en `contacts-routing`:

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

---

En `main.component.html`

```html
<li><a routerLink="contacts">Forms</a></li>
```

---

## 1.1 Directivas

> Para empezar agregamos algunas propiedades. En `contacts.component.ts`:

```typescript
header = 'Contacts';
description = 'Manage your contact list';
numberOfContacts = 0;
counterStyleColor = 'green';
counterClass = 'warning';
formHidden = false;
```

---

### 1.1.1 Enlace del modelo hacia la vista

En `contacts.component.html` mostramos cabeceras con estilo

```html
<h3>
  {{ header }}: {{ description | uppercase }}
</h3>
<p [style.color]="'green'">You have <mark class="{{ counterClass }}">{{ numberOfContacts }}</mark> contacts right now.</p>
```

---

### 1.1.2 Enlace de la vista hacia el modelo

En `contacts.component.html` también actuamos sobre la vista

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

---

# 2. Doble Binding

## NgModel

## Form

### Enlace del modelo hacia la vista

### Enlace de la vista hacia el modelo

---

## 2.1 NgModel

La directiva _ngModel_ viene en el `FormsModule`

Hay que importarlo antes de usarlo. Por ejemplo en `contacts.module.ts`

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

---

### Banana in a box [()]

Hace referencia al _paréntesis dentro del corchete_

```html
[(ngModel)]="model.property"
```

Usa la comunicación en ambos sentidos

- **(banana)** : de la vista al modelo
- **[box]** : del modelo a la vista

--

> La directiva se asocia con una propiedad del controlador...
> o mejor aún, con una **propiedad del modelo** del controlador

---

#### Modelo

```typescript
public contact = { name: '' };
```

#### Directiva

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

```html
<pre>{{ contact | json }}</pre>
```

---

## 2.2 Form

Hay más usos de las directivas en los formularios

Por ejemplo, dado el siguiente modelo:

```typescript
public contact = { name: '', isVIP: false, gender: '' };
```

---

### 2.2.1 CheckBox

```html
<section>
  <label for="isVIP">Is V.I.P.</label>
  <input name="isVIP" type="checkbox" [(ngModel)]="contact.isVIP" />
</section>
```

### 2.2.2 Radio Buttons

```html
<section>
  <label for="gender">Gender</label>
  <input name="gender" value="m" type="radio" [(ngModel)]="contact.gender" />
  <i>Male</i>
  <input name="gender" value="f" type="radio" [(ngModel)]="contact.gender" />
  <i>Female</i>
</section>
```

---

# 3. Estructuras

## \*ngFor

## \*ngIf

---

## 3.1 \*ngFor

Directiva estructural **repetitiva**

Dado el siguiente modelo:

```typescript
public workStatuses = [
  { id: 0, description: 'unknow' },
  { id: 1, description: 'student' },
  { id: 2, description: 'unemployed' },
  { id: 3, description: 'employed' }
];
public contact = { name: '', isVIP: false, gender: '', workStatus: 0 };
```

---

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

> let **iterador** of **iterable**

> > uso avanzado de _trackBy_

---

## 3.2 \*ngIf

Directiva estructural **condicional**

Dado el siguiente modelo

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

> if **condition** else **template**

> > también hay _\*ngSwitch_

---

# 4. Modelo y controlador

## Interfaces y modelos

## ViewModel en el controlador

---

## 4.1 Interfaces y modelos

> Mejor interface que clase

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

---

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

---

## 4.2 ViewModel en el controlador

No solo propiedades, también métodos

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

```html
<input value="Save" type="submit" (click)="saveContact()" />
```

---

### OnInit

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

---

### Repasamos

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



Mira el código completo de **la clase** `ContactsComponent`en el fichero `contacts.component.ts` para tener una visión completa del componente. Como ves, **las propiedades** `header, numContacas, formHidden, contacts ...` se corresponden con las utilizadas en las directivas de enlace en la vista. **Los métodos** `saveContact(), deleteContact()` son invocados desde eventos de elementos del _html_.


Juntos, **la vista y su clase controladora**, resuelven un problema de interacción con el usuario **creando un componente**. Todas las páginas que diseñes serán variaciones y composiciones de estos componentes.

> Y esto es sólo el comienzo. La idea de componente será fundamental en la web del mañana para la creación de páginas mediante `web components`. Pero eso ya se verá más adelante...

Ahora tienes una aplicación en _Angular 9_ que recoge y muestra datos. Sigue esta serie para añadirle [Flujo de datos entre componentes Angular](../flujo-de-datos-entre-componentes-angular/) mientras aprendes a programar con Angular8. Todos esos detalles se tratan en el [curso básico online](https://www.trainingit.es/curso-angular-basico/?promo=angular.builders) que imparto con TrainingIT o a medida para tu empresa.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>
