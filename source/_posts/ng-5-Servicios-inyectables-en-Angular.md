---
title: Servicios inyectables en Angular
permalink: servicios-inyectables-en-Angular
date: 2019-02-15 9:54:58
tags:
  - Angular
  - Servicios
  - DI
  - Tutorial
  - Introducción
  - Angular7
  - Angular2
categories:
  - [Tutorial, Angular]
thumbnail: /css/images/angular-5_inject.png
---

![servicios-inyectables-en-Angular](/images/tutorial-angular-5_inject.png)

La presentación, la lógica y el manejo de datos son tres capas de abstracción que usamos los programadores para mantener organizado nuestro código. En Angular, la presentación es cosa de los componentes. **La lógica y los datos tienen su lugar en servicios compartidos**.

Para que los componentes consuman los servicios de forma controlada tenemos proveedores _inyectables_ en la librería `@angular/core` con los que realizar **la inyección de dependencias**.

<!-- more -->

Partiendo de la aplicación tal cómo quedó en [Flujo de datos entre componentes Angular](../flujo-de-datos-entre-componentes-angular/). Al finalizar tendrás una aplicación que comunica componentes entre páginas, reparte responsabilidades y gestiona claramente sus dependencias.

> Código asociado a este artículo en _GitHub_: [AcademiaBinaria/angular-basic/5-inject](https://github.com/AcademiaBinaria/angular-basic/tree/master/src/app/5-inject/converter)
>
> > Tienes una versión desplegada operativa para probar [Angular Basic](https://academiabinaria.github.io/angular-basic/)

# 1. Inyección de dependencias

Como casi todo en Angular, **los servicios son clases TypeScript**. Su propósito es contener lógica de negocio, clases para acceso a datos o utilidades de infraestructura. Estas clases son perfectamente instanciables desde cualquier otro fichero que las importe. Pero Angular nos sugiere y facilita que usemos su sistema de inyección de dependencias.

Este sistema se basa en convenios y configuraciones que controlan la instancia concreta que será inyectada al objeto dependiente. Ahora verás cómo funciona la **[Dependency Injection](https://es.wikipedia.org/wiki/Inyecci%C3%B3n_de_dependencias) en Angular**.

Como demostración vamos a trabajar con un par de utilidades para conversión de unidades. Crearé un módulo y un componente en el que visualizarlo.

```shell
ng g m 5-inject/converter --routing true
ng g c 5-inject/converter/converter
```

## 1.1 Generación de servicios

La particularidad de las clases de servicios está en su decorador: `@Injectable()`. Esta función viene en el `@angular/core` e **indica que esta clase puede ser inyectada** dinámicamente a quien la demande. Aunque es muy sencillo crearlos a mano, el CLI nos ofrece su comando especializado para crear servicios. Estos son ejemplos de instrucciones para crear un _service_.

```shell
ng g s 5-inject/converter/calculator
```

El resultado es el fichero `calculator.service.ts` con su decorador que toma una _class_ normal y produce algo _injectable_. Veamos una implementación mínima:

```typescript
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root',
})
export class CalculatorService {
  constructor() {}

  public fromKilometersToMiles = kilometers => kilometers * 0.62137;
}
```

Ahora tienes centralizado en este servicio la lógica de datos que tenemos hasta el momento. Los demás componentes la podrán utilizar.

## 1.2 Consumo de dependencias

Declarar y decorar la clase no es suficiente para poder reclamarla. Necesitas **registrarla como un proveedor en algún módulo**. Desde Angular 6 los servicios se auto-proveen en el módulo raíz mediante la configuración `providedIn: 'root'` de su decorador.

> Esto es útil y cómodo en una gran cantidad de casos. El módulo raíz es visible para toda la aplicación de forma que cualquier componente puede reclamar un servicio suyo sin problema. Excepto que el problema sea el tamaño. El módulo raíz se carga al arrancar y todas sus referencias van el _bundle_ principal. Si queremos repartir el peso debemos llevar ciertos servicios al módulo funcional que los necesite.

Vamos a consumir este servicio en el `converter.component.ts`. Al consumo de los servicios inyectables se le conoce como _dependencia_. Cada componente o servicio puede **declarar en su constructor sus dependencias** hacia servicios inyectables. El convenio exige que se especifique el tipo esperado

```typescript
export class ConverterComponent implements OnInit {
  public kilometers = 0;
  public miles: number;

  constructor(private calculatorService: CalculatorService) {}

  public ngOnInit() {
    this.convert();
  }

  public convert() {
    this.miles = this.calculatorService.fromKilometersToMiles(this.kilometers);
  }
}
```

> Agregar el modificador de alcance `private` o `public` en la declaración de argumentos hace que _TypeScript_ genere una propiedad inicializada con el valor recibido. Es azúcar sintáctico para no tener que declarar la propiedad y asignarle el valor del argumento manualmente. En resumen, los [constructores en TypeScrip](https://kendaleiv.com/typescript-constructor-assignment-public-and-private-keywords/) admiten argumentos que transforman en propiedades. Mantenemos privado el `converterService` para evitar su uso desde la vista.

```html
<h2>Distance Converter.</h2>
<h3>From Europe to USA</h3>
<form>
  <fieldset>
    <section>
      <label for="kilometers">Kilometers</label>
      <input name="kilometers" type="number" [(ngModel)]="kilometers" placeholder="0" />
    </section>
  </fieldset>
  <input value="Convert" type="button" (click)="convert()" />
</form>
<section>
  <h4>{{ miles | number:'1.2-2' }} miles</h4>
</section>
```

# 2. Inversión del control

Un concepto íntimamente relacionado con la inyección de dependencias es el de [**Inversion of Control**](https://en.wikipedia.org/wiki/Inversion_of_control). El componente dependiente expresa sus necesidades, pero es el _framework_ el que en última instancia decide lo que recibirá. Vemos entonces que **el invocado cede el control al invocador**.

Cuando proveemos un servicio en Angular, el comportamiento por defecto es el de proveer un _singleton_ pero hay más opciones. Si se usa el objeto `provider` con `useClass` , `useValue` y `useFactory` podemos controlar el proceso de inyección.

Se crea un [_singleton_](https://es.wikipedia.org/wiki/Singleton) por cada módulo en el que se provea un servicio. Normalmente si el servicio es para un sólo módulo funcional se provee en este y nada más. Si va a ser compartido gana la opción de auto proveerlo en el raíz, garantizando así su disponibilidad en cualquier otro módulo de la aplicación.

En un módulo cualquiera, siempre podríamos agregar un servicio a su array de _providers_.

```typescript
@NgModule({
  declarations: [...],
  imports: [...],
  providers: [ CalculatorService ]
})
```

Pero siempre será **una instancia única por módulo**. Si un _singleton_ no es lo adecuado, entonces puedes proveer el mismo servicio en distintos módulos. De esa forma se creará una instancia distinta para cada uno. Si se provee la misma clase en dos o más módulos se genera una instancia en cada uno de ellos. Los componentes recibirán la instancia del módulo jerárquicamente más cercano.

> Incluso es posible usar el array `providers:[]` en la decoración de un componente o de otro servicio. Haciendo así aún más granular la elección de instancia.

Veamos un ejemplo extendiendo el problema del conversor de unidades de forma que se pueda escoger una **estrategia** de conversión en base a una cultura concreta. Para empezar necesitamos una interfaz, un servicio base que la implemente y un componente que lo consuma.

```shell
ng g interface 5-inject/converter/i-culture-converter
ng g service 5-inject/converter/culture-converter
ng g component 5-inject/converter/culture-converter
```

```typeScript
export interface ICultureConverter {
  sourceCulture: string;
  targetCulture: string;
  convertDistance: (source: number) => number;
  convertTemperature: (source: number) => number;
}
```

```typescript
export abstract class CultureConverterService implements ICultureConverter {
  sourceCulture: string;
  targetCulture: string;
  convertDistance: (source: number) => number;
  convertTemperature: (source: number) => number;

  constructor() {}
}
```

```typeScript
export class CultureConverterComponent implements OnInit {
  public source: string;
  public target: string;
  public sourceUnits = 0;
  public targetUnits: number;

  constructor(private cultureConverterService:CultureConverterService){ }

  public ngOnInit() {
    this.source = this.cultureConverterService.sourceCulture;
    this.target = this.cultureConverterService.targetCulture;
    this.convert();
  }
  public convert() {
    this.targetUnits = this.cultureConverterService.convertDistance(this.sourceUnits);
  }
}
```

```html
<h2>Culture Converter.</h2>
<h3>From {{ source }} to {{ target }}</h3>
<form>
  <fieldset>
    <section>
      <label for="sourceUnits">Distance</label>
      <input name="sourceUnits" type="number" [(ngModel)]="sourceUnits" placeholder="0" />
    </section>
  </fieldset>
  <input value="Convert" type="button" (click)="convert()" />
</form>
<section>
  <h4>Distance {{ targetUnits | number:'1.2-2' }}</h4>
</section>
```

## 2.2 Implementaciones

El `CultureConverterComponent` depende de `CultureConverterService` el cual implementa de forma abstracta la interfaz `CultureConverter`. Pero eso no es para nada funcional. Vamos a crear dos implementaciones específicas para Europa y USA. Estas clases concretas se apoyarán en el anteriormente creado `CalculatorService` que necesita algo más de código.

```typescript
export class CalculatorService {
  constructor() {}

  public fromKilometersToMiles = kilometers => kilometers * 0.62137;
  public fromMilesToKilometers = miles => miles * 1.609;
  public fromCelsiusToFarenheit = celsius => celsius * (9 / 5) + 32;
  public fromFarenheitToCelsius = farenheit => (farenheit - 32) * (5 / 9);
}
```

Y aquí están las dos servicios concretos.

```typescript
@Injectable()
export class EuropeConverterService implements ICultureConverter {
  sourceCulture = 'USA';
  targetCulture = 'Europe';

  constructor(private converterService: CalculatorService) {}

  public convertDistance = this.converterService.fromMilesToKilometers;
  public convertTemperature = this.converterService.fromFahrenheitToCelsius;
}
```

```typescript
@Injectable()
export class UsaConverterService implements ICultureConverter {
  sourceCulture = 'Europe';
  targetCulture = 'USA';

  constructor(private converterService: CalculatorService) {}

  public convertDistance = this.converterService.fromKilometersToMiles;
  public convertTemperature = this.converterService.fromCelsiusToFahrenheit;
}
```

## 2.3 Provisión manual

Por ejemplo si queremos utilizar la implementación concreta de USA lo indicamos en el módulo que lo consuma.

```typescript
{
  providers: [
    {
      provide: CultureConverterService,
      useClass: UsaConverterService,
    },
  ];
}
```

El componente reclama una instancia de `CultureConverterService` y le damos otra con la misma interfaz. De esta forma podríamos tener módulos distintos, cada uno con su propia estrategia de conversión.

## 2.4 Factoría

Una situación muy común es poder elegir dinámicamente la implementación concreta. Para ello necesitamos una función factoría que con alguna lógica escoja la estrategia concreta.

```typescript
const cultureFactory = (converterService: ConverterService) => {
  if (environment.unitsCulture === 'metric') {
    return new EuropeConverterService(converterService);
  } else {
    return new UsaConverterService(converterService);
  }
};
{
  providers: [
    {
      provide: CultureConverterService,
      useFactory: cultureFactory,
      deps: [CalculatorService],
    },
  ];
}
```

De esta forma la aplicación se comportará distinto en función de una variable de entorno.

Ya tenemos la aplicación mucho mejor estructurada, pero el almacén de datos es mejorable. Se mantienen los datos _hard-coded_, muy incómodo para actualizar; y en memoria, poco fiable y volátil. Lo más habitual es guardar y recuperar la información en un servidor _http_. Sigue esta serie para añadir [Comunicaciones HTTP en Angular](../comunicaciones-http-en-Angular/) mientras aprendes a programar con Angular.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>
