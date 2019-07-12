---
title: Nx, mono repositorios en Angular
permalink: nx-mono-repositorios-en-Angular
date: 2019-07-11 18:59:27
tags:
- Angular
- Angular8
- Nx
- Tutorial
- Avanzado
categories:
- [Tutorial, Angular]
thumbnail: /css/images/angular-10_monorepo.png
---

![Nx-mono-repositorios-en-Angular](/images/tutorial-angular-10_monorepo.png)

Empiezo este **tutorial de Angular Avanzado** con la frase con la que acabé un artículo de opinión sobre arquitectura de software acerca de [Angular para grandes aplicaciones.](https://medium.com/@albertobasalo71/angular-para-grandes-aplicaciones-b66786fd3032)

> Angular y las decisiones de diseño que le acompañan tienen como objetivo facilitar el desarrollo y mantenimiento a medio y largo plazo de aplicaciones web no triviales.

Las empresas de desarrollo y los clientes finales que escogen **Angular**, suelen ser de tamaño medio o grande. Cuanto mayor sea el problema más destaca este _framework_. Y tarde o temprano esos grandes proyectos necesitarán compartir o reutilizar código. La herramienta [Nx de Nrwl](https://nx.dev/angular) ayuda en esa tarea facilitando la creación de espacios de trabajo multi proyecto: **los mono repositorios.**

<!-- more -->

Partiendo de cero y usando las herramientas de [Nrwl.io/](https://nrwl.io/) crearemos un _blueprint_ para desarrollar grandes aplicaciones. Al finalizar tendrás, en el mismo repositorio, un par de aplicaciones y varias librerías reutilizables creadas con los _Nx power-ups_.

> Código asociado a este tutorial en _GitHub_: [angular.builders/angular-blueprint/](https://github.com/angularbuilders/angular-blueprint)
>
> > Tienes más información sobre este proyecto en [Angular.Builders](https://angular.builders/)

---

# 1. Crear el repositorio

> Como arquitecto de software quiero disponer de un espacio de trabajo único para crear aplicaciones y librerías.

Lo primero será preparar las herramientas. **Nx** es un complemento del **CLI** así que debemos tener este último disponible. Voy a emplear [yarn](https://yarnpkg.com/lang/en/) para la instalación de paquetes y la ejecución de comandos. Pero se muestran las instrucciones alternativas con `npm`. El repositorio siempre lo creo vacío y después agrego las capacidades específicas para **Angular**.

```bash
# Add latest Angular CLI
yarn global add @angular/cli
# Sets yarn as default packager for cli
ng config -g cli.packageManager yarn
# Creates empty repository
yarn create nx-workspace angular-blueprint

# also with NPM...
npm i -g @angular/cli
npx create-nx-workspace@latest angular-blueprint

# Adds Angular capabilities
ng add --dev @nrwl/angular
```

---

# 2. Generar una SPA con Angular

> Como desarrollador Angular quiero tener una aplicación SPA configurada para empezar con una base sólida.

Los próximos comandos te sonarán a los mismo del **angular-cli**. Es normal, pues **Nx** utiliza y mejora las capacidades de la herramienta original. La diferencia está en que la recién creada aplicación, en lugar de nacer en la raíz del _workspace_, va la carpeta específica `/apps`.

```bash
# Generate an Angular application with nx power-ups
ng g @nrwl/angular:application spa --routing=true --style=css --enableIvy=true --prefix=ab-spa --directory=
# Start !!!
yarn start
```

---

# 3. Generar una web simple con Angular

> Como desarrollador Angular quiero tener una aplicación web sencilla para empezar con una base simple.

Sin sorpresas. Usando el mismo generador vamos a crear otra aplicación con sus propias opciones, pero a su vez compartiendo aspectos de su hermana mayor. Por ejemplo `/node_modules`, lo cual se agradece en el tiempo y en el espacio.

También comparten la configuración del `angular.json` y las demás herramientas de ayuda como **tslint** y **prettier**. Ambas, por cierto, vienen completamente configuradas por **Nx**.

```bash
# Generate an Angular application with nx power-ups
ng g @nrwl/angular:application web --routing=false --style=css --enableIvy=true --prefix=ab-web --directory=
# Start !!!
yarn start
```

---

# 4. Tener una biblioteca Angular con componentes propios

> Como desarrollador quiero tener una biblioteca con componentes exportados para que los pueda usar en varias aplicaciones.

Si eres una empresa consultora es posible que te encuentres repitiendo funciones o pantallas una y otra vez para distintos clientes. Por supuesto que una gran empresa seguro que se hacen muchas aplicaciones similares, a las que les vendría de maravilla **compartir una biblioteca de componentes**.

Pues ahora crear librerías es igual de sencillo que crear aplicaciones. **Nx** las depositará en la carpeta `/libs` y se ocupará de apuntarlas en el `tsconfig.json` para que la importación desde el resto del proyecto use alias cortos y evidentes.

Crear componentes en un entorno multi proyecto requiere especificar a qué proyecto se asociarán. PAra empezar vamos a crear los componentes más sencillos posible.

```bash
# Generate an Angular library with nx power-ups
ng g @nrwl/angular:library layout --routing=false --style=css --prefix=ab-layout --directory=
# Generate Header Component
ng g @schematics/angular:component header --project=layout --export=true
# Generate Nav Component
ng g @schematics/angular:component nav --project=layout --export=true
# Generate Footer Component
ng g @schematics/angular:component footer --project=layout --export=true
```
Puedes usarlos como cualquier otro importando el módulo en cualquier aplicación del repositorio.

```TypeScript
import { LayoutModule } from '@angular-blueprint/layout';
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppComponent } from './app.component';

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule, LayoutModule],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

---

# 5. Tener una biblioteca TypeScript con lógica de instrumentación.

> Como arquitecto quiero tener una biblioteca en TypeScript con lógica de instrumentación de modo que pueda usarla con varios frameworks o incluso en puro JavaScript.

Lo primero será crear la librería. Pero esta vez no usaremos los _schematics_ del **cli**, si no los propios de **nrwl**.

```bash
# Generate a Type Script library with nx power-ups
ng g @nrwl/workspace:library tracer --directory=
```
Por ahora no te preocupes de la implementación. La muestro para destacar las dos cosas que considero más importantes:

- No hay ninguna referencia explícita a _Angular_.

- Lo que quieras exportar debe indicarse en el fichero `index.ts`.

`model/trace.js`

```typescript
export type origins = 'api' | 'app' | 'auth' | 'net' | 'system' | 'test' | 'ui';
export type traceTypes = 'business' | 'error' | 'system';
export interface Trace {
  origin: origins;
  type: traceTypes;
  message: string;
  error?: any;
  parameter?: {
    label: string;
    value: number;
  };
}
```

`console-tracer.js`

```typescript
export class ConsoleTracer implements Tracer {
  public writeTrace(trace: Trace) {
    const origin = trace.origin.toLocaleUpperCase();
    const consoleMessage = `[${origin}]: ${trace.message}`;
    switch (trace.type) {
      case 'system':
        console.log(consoleMessage);
        break;
      case 'error':
        console.error(consoleMessage);
        break;
      default:
        break;
    }
    return consoleMessage;
  }
}
```

`index.ts`

```typescript
export * from './lib/model/trace';
export * from './lib/console-tracer';
```

---


# 6. Tener una biblioteca Angular con lógica de instrumentación.

> Como desarrollador quiero tener una biblioteca Angular con servicios de instrumentación para que cualquiera pueda inyectarlos en varias aplicaciones Angular.

La anterior librería es directamente utilizable por cualquier aplicación web, por supuesto incluido **Angular**. Pero, una vez que tengamos la lógica encapsulada en algo reutilizable entre _frameworks_, podemos preparar un módulo específico con servicios para facilitar la inyección de dependencias tradicional de **Angular**.

```bash
# Generate an Angular library with nx power-ups
ng g @nrwl/angular:library angular-tracer --routing=false --style=css --prefix=ab-tracer --directory=
# Generate LoggerService service
ng g @schematics/angular:service consoleTracer --project=angular-tracer
```

Ya que estamos en ambiente **Angular** podemos hacer uso de los productos su ecosistema, como por ejemplo **RxJs**. De esta forma no comprometemos el uso de la librería básica en proyectos que no puedan o quieran usar _observables_.

`console-tracer-service.ts`

```typescript
export class ConsoleTracerService {
  private tracer: ConsoleTracer = new ConsoleTracer();

  constructor() {}

  public subscribe(source: Observable<Trace>) {
    return source.pipe(filter(this.byType))
      .subscribe(this.tracer.writeTrace);
  }
  private byType = (trace: Trace) => trace.type !== 'business';
}
```

Y por supuesto se pueden importar y declarar en cualquier aplicación. Como si fuesen servicios del sistema.

`spa/app.module.ts`

```typescript
@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule,
    RouterModule.forRoot([], { initialNavigation: 'enabled' }),
    LayoutModule,
    AngularTracerModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule {
  constructor(consoleTracerService: ConsoleTracerService) {
    const startMessage: Trace = {
      origin: 'system',
      type: 'system',
      message: 'App Module Started'
    };
    consoleTracerService.subscribe(of(startMessage)).unsubscribe();
  }
}
```

---

Completaré este articulo con algo que no puede faltar en ningún proyecto profesional, las pruebas. De nuevo la gente de *Nrwl* ha pensado en ello y el **Nx** instala y configura dos productos de última generación que facilitan la tarea. Usaremos [Jest](https://jestjs.io/) para los test unitarios y [Cypress](https://www.cypress.io/) para los de integración _End to End_.

Con este conocimiento empiezas tu formación [avanzada en Angular](../tag/Avanzado/).

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>
