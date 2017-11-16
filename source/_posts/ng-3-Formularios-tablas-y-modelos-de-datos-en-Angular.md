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
**Los formularios son el punto de entrada** de información a nuestros sistemas. Llevan con nosotros desde el inicio de la propia informática y se han comido una buena parte del tiempo de programación. En *Angular* han prestado una atención a ellos facilitando su desarrollo, **desde pantallas simples a complejos procesos**.

## 1.1 El Binding
La clave para entender cómo funciona *Angular* está en el concepto de **enlace entre elementos html de las vistas y propiedades de modelos** de datos, el llamado `binding`.


### 1.1.1 La interpolación entre dobles llaves 
Ya hemos visto [ejemplos de binding](https://github.com/AcademiaBinaria/angular5/blob/master/3-data/cash-flow/src/app/lib/components/nav/title.component.ts) sencillos en este tutorial. 
En el fichero `new.component.ts` tienes en su vista html la directiva `{{ title | uppecarse }}`. Esas dobles llaves encierran expresiones que se evaluarán en tiempo de ejecución. La llamamos **directiva de interpolación** y es la manera más cómoda y usual de mostrar contenido dinámico en Angular.

```typescript
@Component({
  template: `
    <h2>{{ title | uppercase }}</h2>`
})
export class NewComponent implements OnInit {
  title = "Cash Flow";
  constructor() {}
  ngOnInit() {}
}
```

>La expresión interna hace referencia a variables que se obtienen de las propiedades de la clase controladora del componente. En este caso `NewComponent` y `title`, con su valor *New Operation* en ejecución.  Este enlace mantiene la vista permanentemente actualizada a través de un potente sistema de detección del cambio. 

### 1.1.2 Las tuberías |
Si queremos que la presentación del dato sea distinta a su valor real, podemos usar **funciones de transformación** especiales. Se llaman tuberías o *pipes* y se indican mediante el carácter `|`.

El *framework* nos provee de casos básicos como `uppercase, lowercase, date, number...`. También dispones de un mecanismo para crear tus propios *pipes*.

>En el caso anterior verás en ejecución el texto *NEW OPERATION*

## 1.2 Doble Binding
La comunicación del modelo hacia la vista es sólo el principio. En *Angular* también podrás **comunicar la vista hacia el modelo**, permitiéndole al usuario modificar los datos a través de formularios. En el fichero `new.component.ts` tienes un ejemplo; vamos a analizarlo:

```html
<form class="container">
  <label for="description">Description</label>
  <input name="description"
        #inputDescription
        [value]="operation.description"
        (change)="operation.description=inputDescription.value"
        type="text" />
  <label for="amount">Amount</label>
  <input name="amount"
        [(ngModel)]="operation.amount"
        type="number"/>
  <label>Kind of Operation</label>
  <select name="kind" [(ngModel)]="operation.kind">
    <option [value]="">Please select a kind</option>
    <option *ngFor="let kind of kindsOfOperations"
          [value]="kind">{{kind}}</option>
  </select>
  <button (click)="saveOperation()">Save</button>
</form>
```
### 1.2.1 Identificadores con #
En el código anterior apreciaras que junto a elementos del estándar del *html* aparecen signos extraños. Por ejemplo en la primera etiqueta `input` aparece un atributo llamado `#inputDescription`. A estos atributos proporcionados por Angular les llamaremos **directivas**. La directiva `#` genera un indentificador único para el elemento al que se le aplica y permite referirse a él en otros lugares del código.

### 1.2.2 Propiedades entre []
Un viejo conocido como el atributo `value` recibe habitualmente un valor concreto, una constante. Pero, si se encierra entre corchetes se convierte en un **evaluador de expresiones** y puede recibir una variable. En este caso `[value]="operation.description"` asigna el valor de esa expresión del modelo al elemento html.

### 1.2.2 Eventos entre ()
Los eventos del html llevan años entre nosotros. En Angular se expresan de una manera distinta, encerrándolos entre paréntesis. **Los eventos reciben una instrucción a ejecutar** cuando el usuario dispare el detonante. Aquí `(change) = "operation.description = inputDescription.value"` se usa para guardar en el modelo el valor actual del elemento *input* ante cada cambio en este. Consiguiendo así el doble binding.

### 1.2.3 Modelos entre [()]
El patrón anterior podrías replicarlo una y otra vez. Pero en *Angular* te ofrecen un atajo para estos casos; es la directiva `[(ngModel)]`. Esta directiva también es conocida como *banana in a box* porque su sintaxis requiere un `()` dentro de un `[]`. 
Por ejemplo `[(ngModel)]="operation.amount"` enlaza doblemente el modelo `operation.amount` con la el elemento `input` de la vista. 


# 2 Las repeticiones
Una situación que nos encontramos una y otra vez es la de las repeticiones. Listas de datos, tablas o grupos de opciones son ejemplos claros. Hay una directiva en *Angular* para esa situación, la `*ngFor="let iterador of array"`. **La directiva `*ngFor` forma parte del grupo de directivas estructurales**, llamadas aís porque modifican la estructura del DOM, en este caso insertando múltiples nodos hijos a un elemento dado.

>Puedes ver un ejemplo del uso la directiva `*ngFor` en el elemento `select`. Se emplea para recorrer un array y generar a partir de sus valores el grupo de potenciales opciones para el usuario. 

Pero el caso uso *más repetido de las repeticiones* es el de mostrar tablas o listas de datos.

## 2.1 Tablas
La aplicación del ejemplo tiene un formulario que, aún no te he explicado cómo, guarda el trabajo del usuario en un array. Ese mismo **array se muestra como una tabla** de datos valiéndose de `*ngFor`. Para montar una tabla sólo necesito un código como este:

```html
<table *ngIf="numberOfOperations>0;else emptyList">
  <thead>
    <tr>
      <th>Description</th>
      <th>Kind</th>
      <th>Amount</th>
      <th>Delete</th>
    </tr>
  </thead>
  <tbody>
    <tr *ngFor="let operation of operations">
      <td>{{ operation.description }}</td>
      <td>{{ operation.kind }}</td>
      <td>{{ operation.amount | number:'7.2-2' }}</td>
      <td><button (click)="deleteOperation(operation)">Delete</button> </td>
    </tr>
  </tbody>
</table>
<ng-template #emptyList>
  <h3>No operations yet.</h3>
</ng-template>
```

>Todo lo aquí presente son directivas ya conocidas. La famosa directiva estructural `*ngFor="let operation of operations"`.  Las interpolaciones con tuberías como `operation.amount | number:'7.2-2'`. La subscripción a eventos de `(click)="deleteOperation(operation)"`.

## 2.2 Condicionales 
Otra directiva estructural muy utilizada es la `*ngIf`, la cual consigue que un elemento se incluya o se elimine en el *DOM* en función de los datos del modelo. 

>En el ejemplo puedes ver que la uso para mostrar la tabla sólo si tiene registrso. En otro aparecerá el mensaje de *No operations yet.* 

Todas estas directivas permiten crear interfaces de usuario dinámicas y conducidas por los datos. Es hora de que veas cómo manejar esos datos.

# 3 Modelo y controlador
Los componentes los hemos definido como **bloques de constucción de páginas. Mediante una vista y un controlador** resuelven un problema de interación o presentación de modelos. En los puntos anteriores te presenté la vista. Toca ahora estudiar el modelo y el controlador.

## 3.1 El modelo 
Sin ir muy lejos en las capacidades que tendría un modelo de datos clásico, vamos al menos a beneficiarnos del ***TypeScript* para definir la estrucutra de datos**. Esto facilitará la programación mediante el autocompletado del editor y reducirá los errores de tecleo mediante la comprobación estática de tipos.

Para ello necesito una clase sencilla, que bien se podría crear a mano. Pero te recomiendo que sigas familiarizándote con las capacidades de generación de código del *CLI* y uses el siguiente comando:

```shell
ng g class /views/operations/operation
```

En el fichero resultado `operation.ts` he metido una definción de clase simple pero que muestra algunas de las capacidades del *TypeScript*.

```typescript
export class Operation {
  public _id: string;
  public amount: number = 0;
  public description: string = "";
  public kind: string;
}
```
Como te digo, este fichero sólo aporta estructura a los datos. Más adelante te contaré dónde codificar los métodos de manejo de datos.

## 3.2 El controlador 
La parte de **lógica del componente** va en la clase que se usa oara us definción. Como [ya has visto](https://github.com/AcademiaBinaria/angular5/blob/master/3-data/cash-flow/src/app/views/operations/item.component.ts) podemos usar su constructor para reclamar dependencias y usar los interfaces para responder a eventos de su ciclo de vida. Ahora se trata de crear propiedades y métodos con los que comunicarse con la vista.

Podemos decir que las propiedades públicas de la clase actuarán como *binding* de datos con la vista. Mientras que los métodos públicos serán invocados desde los eventos de la misma vista.

Mira el código de la clase `NewComponent`en el fichero `new.component.ts`:

```typescript
export class NewComponent implements OnInit {
  public kindsOfOperations = ["Income", "Expense"];
  public numberOfOperations = 0;
  public operation: Operation = new Operation();
  public operations: Operation[] = [];

  constructor() {}
  ngOnInit() {}
  
  public saveOperation() {
    const clonedOperation = this.cloneOperation(this.operation);
    clonedOperation._id = new Date().getTime().toString();
    this.operations.push(clonedOperation);
    this.numberOfOperations = this.operations.length;
    this.operation = new Operation();
  }
  public deleteOperation(operation: Operation) {
    const index = this.operations.indexOf(operation);
    this.operations.splice(index, 1);
    this.numberOfOperations = this.operations.length;
  }
  cloneOperation(originalOperation: Operation): Operation {
    const targetOperation = Object.assign({}, originalOperation);
    return targetOperation;
  }
}
```
Como ves, las propidades `kindsOfOperations, numberOfOperations, operation y operations` se corresponden con las utilizadas en las directivas de enlace en la vista.

Los métodods `saveOperation() y deleteOperation(operation: Operation)` son invocados desde eventos de elementos del html.

Juntos, **la vista y su clase controladora**, resuelven un problema de interacción con el usuario **creando un componente**. Todas las páginas que diseñes serán variaciones y composiciones de estos componentes. 


Ya tienes una aplicación en *Angular* que recoge y muestra datos. Sigue esta serie para añadirle funcionalidad mientras aprendes a programar con Angular5.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite> 