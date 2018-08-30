---
title: Formularios, tablas y modelos de datos en Angular
permalink: formularios-tablas-y-modelos-de-datos-en-angular
date: 2018-08-29 19:17:37
tags:  
- Angular
- Forms
- Tutorial
- Introducción
categories:
- [Tutorial, Angular] 
thumbnail: /css/images/angular-3_data.png
---
![Tutorial Angular 3-Data](/images/tutorial-angular-3_data.png)

Las **aplicaciones Angular 6 son excelentes para el tratamiento de datos** en el navegador. La recogida de información mediante formularios y la presentación de páginas dinámicas fue su razón de ser.

Vamos a ver cómo la librería `@angular/forms` enlaza **las vistas, los controladores y los modelos** y cómo se hace la presentación de datos en **listas y tablas**.

<!-- more -->

Partiendo de la aplicación tal cómo quedó en [Páginas y rutas Angular SPA](../paginas-y-rutas-angular-spa/). Al finalizar tendrás una aplicación que recoge y presenta datos.

>Código asociado a este artículo en *GitHub*: [AcademiaBinaria/AutoBot/3-data](https://github.com/AcademiaBinaria/autobot/tree/3-data) 







# 1. Formularios

**Los formularios son el punto de entrada** de información a nuestros sistemas. Llevan con nosotros desde el inicio de la propia informática y se han comido una buena parte del tiempo de programación. En *Angular* han prestado una atención a ellos facilitando su desarrollo, **desde pantallas simples a complejos procesos**.

## 1.1 El Binding

La clave para entender cómo funciona *Angular* está en el concepto de **enlace entre elementos html de las vistas y propiedades de modelos** de datos, el llamado `binding`.


### 1.1.1 La interpolación entre  \{ \{ \} \}

En el fichero `car.component.ts` tienes en su vista *html* encontrarás elementos ajenos al lenguaje. Son las directivas. La primera que encuentras es `{{ title }}`. Esas dobles llaves encierran expresiones que se evaluarán en tiempo de ejecución. La llamamos **directiva de interpolación** y es la manera más cómoda y usual de mostrar contenido dinámico en Angular.

Este es un ejemplo en su forma más simple posible.

```typescript
@Component({
  selector: 'app-root',
  template: `<p>{{ title }}</p>`
})
export class AppComponent {
  public title = "Learning Angular";
}
```

>La expresión interna hace referencia a variables que se obtienen de las propiedades de la clase controladora del componente. En este caso `AppComponent` y `title`, con su valor *Learning Angular* en ejecución.  Este enlace mantiene la vista permanentemente actualizada a través de un potente sistema de detección del cambio. 

### 1.1.2 Las tuberías |

Si queremos que la presentación del dato sea distinta a su valor real, podemos usar **funciones de transformación** especiales. Se llaman tuberías o *pipes* y se indican mediante el carácter `|`.

El *framework* nos provee de casos básicos como `uppercase, lowercase, date, number...`. También dispones de un mecanismo para crear tus propios *pipes*.

Más ejemplos los encontrarás en el componente `car.component.ts`. Haciendo uso de los *pipes* `currency` y `number` contra el objeto *car*. 

```html
<div>
    {{ car.model | uppercase }}
  <span>{{ car.cost | currency:'EUR' }}</span>
  <span>{{ car.currentSpeed | number:'1.0-0' }}</span>
</div>
```

### 1.1.3 Los atributos evaluados []

En *Html* disponemos de atributos para asignar valores a propiedades de los elementos. Esos atributos reciben los valores como constantes. Pero, si se encierran entre corchetes se convierten en un **evaluador de expresiones** y puede recibir una variable o cualquier otra expresión. 

Como por ejemplo usando una *porgress bar* cuyo valor cambia en tiempo de ejcución. O para desabilitar un elemento dinámicamente.

```html
<progress [value]="car.currentSpeed" [max]="car.topSpeed"></progress>
<button [disabled]="this.car.currentSpeed <= 0">Brake</button>
```

### 1.1.4 Las clases CSS como atributos especiales

Para el caso concreto de deteminar las clases CSS aplicables a un elemento de manera dinámica, usaremos la directiva `ngClass`. La cual recibe un array de textos, o de variables de tipo *string*, que en ejecución montan la cadena multivaluada de clases CSS a asignar.

```html
<progress [ngClass]="['progress', speedClass]"></progress>
```

Con su código relacionado:

```typescript
if (speedRate >= environment.dangerSpeedRate) {
  this.speedClass = 'is-danger';
} else if (speedRate >= environment.warningSpeedRate) {
  this.speedClass = 'is-warning';
} else {
  this.speedClass = 'is-info';
}
```

### 1.1.5 Los eventos ()

Cualquier evento asociado a un elemento puede ejecutar una instrucción  sin más que incluirlo entre paréntesis. Idealmente dicha instrucción debe llamar a un método o función de la clase controladora.

```html
<button (click)="car.speed = car.speed - 1">Brake</button>
<button (click)="onThrottle()">Throttle</button>
```

## 1.2 Doble Binding: Recepcion y validación

La comunicación del modelo hacia la vista es sólo el principio. En *Angular* también podrás **comunicar la vista hacia el modelo**, permitiéndole al usuario modificar los datos a través de formularios. Es lo que se conoce como *double binding*.

### 1.2.1 El doble enlace al modelo [(ngModel)]

La directiva `[(ngModel)]` se compone de un atributo *custom* `ngModel` y lo rodea de los símbolos `[()]`. Esta técnica es conocida como *banana in a box* porque su sintaxis requiere un `()` dentro de un `[]` y une las capacidades de las expresiones y los eventos facilitando la comunicación bidireccional.

> Atención: La directiva `ngModel` viene dentro del módulo `FormsModule` que hay que importar explícitamente.

Por ejemplo `[(ngModel)]="rechargedDistance"` enlaza doblemente la propiedad del modelo `rechargedDistance` con el elemento `<input>` de la vista. Cada tecleo del usuario se registra en la variable. Y el valor de la variable se muestra en el `<input>`.

```html
<form (ngSubmit)="onRecharge()">
 <input [(ngModel)]="rechargedDistance" name="rechargedDistance">
 <button type="submit">Recharge</button>
</form>
```

> La directiva `ngModel` es mucho más potente de lo visto aquí. Entre otras cosas permite decidir el citerio de actualizción (a cada cambio o al salir del control). También se verá más adelante el asunto de la validación, que requiere un trato especial. 

# 2 Estructuras

Los anteriores modificadores actúan a nivel de contenido del HTML. Veremos ahora una para de directivas que afectan directamente a la estructuradel árbol DOM. Son las llamadas directivas estructurales que comienzan por el signo `*` 

## 2.2 Condicionales \*ngIf

La directiva estructural más utilizada es la `*ngIf`, la cual consigue que un elemento se incluya o se elimine en el *DOM* en función de los datos del modelo. 

>En el ejemplo puedes ver que la uso para mostrar los mandos de acelaración y freno sólo cuando hay batería. En otro aparecerá el formulario de recarga. 

```html
<section *ngIf="hasBattery(); else rechargingSection"  >
  <button (click)="onBrake()">Brake</button>
  <button (click)="onThrottle()">Throttle</button>
</section>
<ng-template #rechargingSection>
  <form (ngSubmit)="onRecharge()" >
    <input [(ngModel)]="rechargedDistance" name="rechargedDistance">
    <button type="submit">Recharge</button>
  </form>
</ng-template>
```
### 2.2.1 Identificadores con hashtag 

En el código anterior apreciarás que aparece un elemento `<ng-template>` no estándar con el atributo llamado `#rechargingSection` precedido por un `#`. La directiva `#` genera un indentificador único para el elemento al que se le aplica y permite referirse a él en otros lugares del código.

Ese truco permite que `*ngIf` muestre otro elemento cuando la condicióon principal falle. El otro elemento tiene que ser el componente especial `<ng-template>` y para localizarlo se usa el identificador `#`.


## 2.2 Repetitivas

Una situación que nos encontramos una y otra vez es la de las repeticiones. Listas de datos, tablas o grupos de opciones son ejemplos claros. Hay una directiva en *Angular* para esa situación, la `*ngFor="let iterador of array"`. **La directiva `*ngFor` forma parte del grupo de directivas estructurales**, porque modifica la estructura del DOM, en este caso insertando múltiples nodos hijos a un elemento dado.

>Puedes ver un ejemplo del uso la directiva `*ngFor` en el componente `HomeComponent`. Se emplea para recorrer un array y una lista de enlaces. Es el caso de uso *más repetido de las repeticiones*; mostrar tablas o listas de datos.

```html
<ul>
  <li *ngFor="let car of cars">
    <a [routerLink]="['/car', car.link.url]">
      <strong>
        {{ car.link.caption }}
      </strong>
      <span>
        {{ car.cost | currency:'EUR' }}
      </span>
    </a>
  </li>
</ul>
```

# 3 Modelo y controlador

Los componentes los hemos definido como **bloques de constucción de páginas. Mediante una vista y un controlador** resuelven un problema de interación o presentación de modelos. En los puntos anteriores te presenté la vista. Toca ahora estudiar el modelo y el controlador.

## 3.1 El modelo y su interfaz

Sin ir muy lejos en las capacidades que tendría un modelo de datos clásico, vamos al menos a beneficiarnos del ***TypeScript* para definir la estructura de datos**. Esto facilitará la programación mediante el autocompletado del editor y reducirá los errores de tecleo mediante la comprobación estática de tipos.

Para ello necesito una interfaz sencilla. Esto es puro *TypeScript*, no es ningún artificio registrable en Angular. Esos sí, en algún sitio tienen que estar. Yo suelo usar la ruta `core/store/models`, pero es algo completamente arbitrario.

```typescript
export interface Car {
  model: string;
  cost: number;
  topSpeed: number;
  currentSpeed: number;
  totalBattery: number;
  remainingBattery: number;
  distanceTraveled: number;
  link: Link;
}
export interface Link {
  url: string;
  caption: string;
}
```

>Te recomiendo que **no uses clases para definir modelos** a menos que necesites agregarle funcionalidad imprescindible. Las interfaces, ayudan al control de tipos en tiempo de desarrollo, igual que las clases, pero sin generar nada de código en tiempo de ejecución, al contrario que las clases.

## 3.1.1 Constantes

En ocasiones, habrá datos *hard-coded* en tu aplicación. En ese caso se resuelve exportando constantes. Por ejemplo en el fichero `core/store/cars.ts` tenemos:

```typescript
import { Car } from './models/car.model';
export const CARS: Car[] = [
  {
    ...
  }
];
```


## 3.1.2 Configuración

Un tipo especial de constantes son aquellas consideradas de configuración. Incluso con valores distintos según el entorno de ejecución. En Angular 6 nos ofrecen la carpeta especial `environments`.

En ella habrá al menos un fichero maestro usado en tiempo de desarrollo y llamado simplemente `environment.ts`. Dentro aparece una única constante exportada con el redundante nombre `environment`. En ese objeto sin esquema predefinido puedes incluir todos los datos de configuración que necesites en tu programa.

```typescript
export const environment = {
  production: false,
  title: 'autobot',
  version: '3-data',
  tag: '3.0.0',
  refreshInterval: 2000,
  dangerSpeedRate: 0.9,
  warningSpeedRate: 0.7,
  dangerKmsBattery: 100,
  warningKmsBattery: 150
};
```

> Además del *master* tenemos al menos otro fichero para el entorno de producción. Pueden crearse más para más entornos, pero este siempre aparece; es el `environment.prod.ts`. Su estructura ha de ser igual a la del master pero al con al menos un cambio en su contenido; `production:true`. Por lo demás el CLI se encarga de compilar la aplicación con los valores tomados del fichero adecuado para el entorno. 

## 3.2 El controlador

La parte de **lógica del componente** va en la clase que se usa para su definción. Como ya has visto podemos usar su constructor para reclamar dependencias y usar los interfaces para responder a eventos de su ciclo de vida. Repasemos el `CarComponent` viéndolo comm la clase que és: en su inicialización, propiedades y métodos.


```typescript
export class CarComponent implements OnInit {
  constructor(private route: ActivatedRoute) {}
  ngOnInit() {
    const carId = this.route.snapshot.params['carId'];
    this.car = CARS.find(c => c.link.url === carId);
    setInterval(() => this.timeGoesBy(), environment.refreshInterval);
  }
}
``` 

Ahora se trata de crear propiedades para aportar datos a la vista

```typescript
export class CarComponent implements OnInit {
  public car: Car;
  public speedClass = 'is-info';
  public batteryClass = 'is-success';
  public rechargedDistance;
  ...
}
``` 

Y los métodos a los que llamará la vista, según actúe el usuario.

```typescript
public onBrake() {
  this.car.currentSpeed--;
}
public onThrottle() {
  this.car.currentSpeed++;
}
public onRecharge() {
  this.car.remainingBattery = this.rechargedDistance;
}
``` 


Podemos decir que las propiedades públicas de la clase actuarán como *binding* de datos con la vista. Mientras que los métodos públicos serán invocados desde los eventos de la misma vista.

Mira el código completo de **la clase** `CarComponent`en el fichero `car.component.ts` para tener una visión completa del componente. Como ves, **las propidades** `car, speedClass, batteryClass,rechargedDistance` se corresponden con las utilizadas en las directivas de enlace en la vista. **Los métodos** `onBrake(), onThrottle(), onRecharge()` son invocados desde eventos de elementos del *html*.

Juntos, **la vista y su clase controladora**, resuelven un problema de interacción con el usuario **creando un componente**. Todas las páginas que diseñes serán variaciones y composiciones de estos componentes. 

> Y esto es sólo el comienzo. La idea de componente será fundamental en la web del mañana para la creación de páginas mediante `web components`. Pero eso ya se verá más adelante...

Ahora tienes una aplicación en *Angular 6* que recoge y muestra datos. Sigue esta serie para añadirle [Flujo de datos entre componentes Angular](../flujo-de-datos-entre-componentes-angular/) mientras aprendes a programar con Angular6.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite> 