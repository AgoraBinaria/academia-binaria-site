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

![nx-mono-repositorios-en-Angular](/images/tutorial-angular-10_monorepo.png)

Empiezo este **tutorial de Angular Avanzado** con la frase con la que acabé un artículo de opinión sobre arquitectura de software acerca de [Angular para grandes aplicaciones.](https://medium.com/@albertobasalo71/angular-para-grandes-aplicaciones-b66786fd3032)

> Angular y las decisiones de diseño que le acompañan tienen como objetivo facilitar el desarrollo y mantenimiento a medio y largo plazo de aplicaciones web no triviales.

Las empresas de desarrollo y los clientes finales que escogen **Angular**, suelen ser de tamaño medio o grande. Cuanto mayor sea el problema más destaca este _framework_. Y tarde o temprano esos grandes proyectos necesitarán compartir o reutilizar código. La herramienta [Nx de Nrwl](https://nx.dev/angular) ayuda en esa tarea facilitando la creación de espacios de trabajo multi proyecto: **los mono repositorios.**

<!-- more -->

Partiendo de cero y usando las herramientas de [Nrwl.io/](https://nrwl.io/) crearemos un _blueprint_ para desarrollar grandes aplicaciones. Al finalizar tendrás, en el mismo repositorio, un par de aplicaciones y varias librerías reutilizables creadas con los _Nx power-ups_.

> Código asociado a este tutorial en _GitHub_: [angular.builders/angular-blueprint/](https://github.com/angularbuilders/angular-blueprint)
>
> > Tienes más información sobre este proyecto en [Angular.Builders](https://angular.builders/)

**Tabla de Contenido: **

[1. Crear el repositorio.](./#1-Crear-el-repositorio)

[2. Generar una SPA con Angular.](./#2-Generar-una-SPA-con-Angular)

[3. Generar una web simple con Angular.](./#3-Generar-una-web-simple-con-Angular)

[4. Tener una biblioteca Angular con componentes propios.](./#4-Tener-una-biblioteca-Angular-con-componentes-propios)

[5. Tener una biblioteca TypeScript con lógica de instrumentación.](./#5-Tener-una-biblioteca-TypeScript-con-logica-de-instrumentacion)

[6. Tener una biblioteca Angular con lógica de instrumentación.](./#6-Tener-una-biblioteca-Angular-con-logica-de-instrumentacion)

[7. e2e: Dada una SPA en Angular.](./#7-e2e-Dada-una-SPA-en-Angular)

[8. e2e: Dada una página web en Angular.](./#8-e2e-Dada-una-pagina-web-en-Angular)

[9. e2e: Dada una librería en Angular con componentes de diseño.](./#9-e2e-Dada-una-libreria-en-Angular-con-componentes-de-diseno)

[10. unit: Dada una librería TypeScript con servicios.](./#10-unit-Dada-una-libreria-TypeScript-con-servicios)

[11. unit: Dada una librería Angular con servicios de instrumentación.](./#11-unit-Dada-una-libreria-Angular-con-servicios-de-instrumentacion)

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
ng g application spa --routing=true --style=css --enableIvy=true --prefix=ab-spa --directory=
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
ng g application web --routing=false --style=css --enableIvy=true --prefix=ab-web --directory=
# Start !!!
yarn start
```

---

# 4. Tener una biblioteca Angular con componentes propios

> Como desarrollador quiero tener una biblioteca con componentes exportados para que los pueda usar en varias aplicaciones.

Si eres una empresa consultora es posible que te encuentres repitiendo funciones o pantallas una y otra vez para distintos clientes. Por supuesto que una gran empresa seguro que se hacen muchas aplicaciones similares, a las que les vendría de maravilla **compartir una biblioteca de componentes**.

Pues ahora crear librerías es igual de sencillo que crear aplicaciones. **Nx** las depositará en la carpeta `/libs` y se ocupará de apuntarlas en el `tsconfig.json` para que la importación desde el resto del proyecto use alias cortos y evidentes.

Crear componentes en un entorno multi proyecto requiere especificar a qué proyecto se asociarán. Para empezar vamos a crear los componentes básicos para cualquier _layout_ lo más sencillos posible.

```bash
# Generate an Angular library with nx power-ups
ng g library layout --routing=false --style=css --prefix=ab-layout --directory=
# Generate Header Component
ng g c components/header --project=layout --export=true
# Generate Nav Component
ng g c components/nav --project=layout --export=true
# Generate Footer Component
ng g c components/footer --project=layout --export=true
```
Puedes usarlos como cualquier otro componente y en cualquier aplicación del repositorio. Simplemente importando el módulo en el que se declaran: el `LayoutModule`. NX se encarga de referenciar cada proyecto en el fichero `tsconfig.json`. De esa forma se facilita su importación.

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

Lo primero será crear la librería. Pero esta vez no usaremos los _schematics_ del **cli**, si no los propios de **nrwl**. La idea es usarla como la **capa de dominio de la arquitectura**. En ella pondremos los modelos y servicios de lógica de negocio con las menores dependencias posibles. En concreto no dependeremos de Angular, lo cual permitiría usarlo con otros _frameworks_ actuales o futuros.

```bash
# Generate a Type Script library with nx power-ups
ng g @nrwl/workspace:library tracer-domain --directory=
```
Por ahora no te preocupes de la implementación. La muestro para destacar las dos cosas que considero más importantes:

- No hay ninguna referencia explícita a _Angular_.

- Lo que quieras exportar debe indicarse en el fichero `index.ts`.

Por lo demás es puro _TypeScript_; en dos carpetas con intenciones bien claras: `models/` y `services/` creo de forma manual los siguientes ficheros:

`models/trace.interface.js`

```typescript
import { traceLevels } from './trace-levels.type';
import { traceOrigins } from './trace-origins.type';

export interface Trace {
  origin: traceOrigins;
  level: traceLevels;
  message: string;
  error?: any;
  parameter?: {
    label: string;
    value: number;
  };
}
```

`services/console-tracer.driver.js`

```typescript
import { Trace } from '../models/trace.interface';
import { Tracer } from '../models/tracer.interface';

export class ConsoleTracerDriver implements Tracer {
  public writeTrace(trace: Trace) {
    switch (trace.level) {
      case 'system':
        return this.writeSystem(trace);
      case 'error':
        return this.writeError(trace);
      default:
        return '';
    }
  }
  public writeSystem(trace: Trace) {
    const origin = this.getOriginPart(trace);
    const consoleMessage = `${origin}${trace.message}`;
    console.log(consoleMessage);
    return consoleMessage;
  }
  public writeError(trace: Trace) {
    const origin = this.getOriginPart(trace);
    const consoleMessage = `${origin}Error`;
    console.group(consoleMessage);
    console.log(trace.message);
    if (trace.error) {
      console.warn(trace.error.message);
      console.log(trace.error.stack || 'no stack');
    }
    console.groupEnd();
    return consoleMessage;
  }

  private getOriginPart = (trace: Trace): string =>
    `[${trace.origin.toLocaleUpperCase()}]: `;
}
```

`index.ts`

```typescript
export * from './lib/models/trace.interface';
export * from './lib/models/tracer.interface';
export * from './lib/services/console-tracer.driver';
```

---


# 6. Tener una biblioteca Angular con lógica de instrumentación.

> Como desarrollador quiero tener una biblioteca Angular con servicios de instrumentación para que cualquiera pueda inyectarlos en varias aplicaciones Angular.

La anterior librería es directamente utilizable por cualquier aplicación web, por supuesto incluido **Angular**. Pero, una vez que tengamos la lógica encapsulada en algo reutilizable entre _frameworks_, podemos preparar un módulo específico con servicios para facilitar la inyección de dependencias tradicional de **Angular**.

```bash
# Generate an Angular library with nx power-ups
ng g library tracer --routing=false --style=css --prefix=ab-tracer --directory=
# Generate a Tracer Service
ng g s services/tracer --project=tracer
# Generate an Error Handler Service
ng g s services/error-handler --project=tracer
```

Ya que estamos en ambiente **Angular** podemos hacer uso de los productos su ecosistema, como por ejemplo _@Inject()_. De esta forma no comprometemos los servicios de la librería con su configuración; la cual vendrá desde la aplicación. Incluso queda preparado para que las clases de dominio o los drivers y repositorios puedan ser inyectados.

En esta caso empezaremos con un objeto con un mísero _Boolean_ para indicarnos si estamos o no en producción. Usaremos la consola para _tracear_ sólo en desarrollo. Por ahora no haremos nada en producción.

`servicers/tracer-service.ts`

```typescript
import { ConsoleTracerDriver, Trace, Tracer } from '@angular-blueprint/tracer-domain';
import { Inject, Injectable, InjectionToken } from '@angular/core';

export interface TracerConfig {
  production: boolean;
}

export const TRACER_CONFIG = new InjectionToken<TracerConfig>('tracer-config');

class NoTrace implements Tracer {
  writeTrace = (trace: Trace) => '';
}

@Injectable({
  providedIn: 'root'
})
export class TracerService implements Tracer {
  private tracer: Tracer;

  constructor(@Inject(TRACER_CONFIG) tracerConfig?: TracerConfig) {
    if (tracerConfig.production) this.tracer = new NoTrace();
    else this.tracer = new ConsoleTracerDriver();
  }

  public writeTrace(trace: Trace): string {
    return this.tracer.writeTrace(trace);
  }
}
```

Y por supuesto se pueden importar y declarar en cualquier aplicación. Como si fuesen servicios del sistema. La magia de la inversión del control se produce con `useValue` mediante el cual inyectamos un valor de configuración concreto.

`spa/app.module.ts`

```typescript
@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule,
    RouterModule.forRoot(appRoutes, { initialNavigation: 'enabled' }),
    LayoutModule,
    TracerModule
  ],
  providers: [
    {
      provide: TRACER_CONFIG,
      useValue: { production: environment.production }
    }
  ],
  bootstrap: [AppComponent]
})
export class AppModule {
  constructor(tracerService: TracerService) {
    const startMessage: Trace = {
      origin: 'system',
      level: 'system',
      message: 'App Module Started for SPA'
    };
    tracerService.writeTrace(startMessage);
  }
}
```
Y ya tenemos un germen de arquitectura flexible (controlada por la inyección de dependencias) y reutilizable (entre aplicaciones Angular) con un dominio estable e independiente de _frameworks_.

Para más ejemplos mira [la implementación un _ErrorHandler_ en el repositorio](https://github.com/angularbuilders/angular-blueprint/blob/master/libs/tracer/src/lib/services/error-handler.service.ts). Es un servicio que una vez proveído hace uso del servicio de trazas.

---


# 7. e2e: Dada una SPA en Angular.

> Dada una SPA en Angular cuando se visita la página de inicio entonces debería mostrar un enlace con el nombre de la aplicación y luego debería mostrar el logotipo

Las pruebas de software..., todos las hacemos ¿no es cierto?. Pues se van acabando las excusas para algo que no puede faltar en ningún proyecto profesional. De nuevo la gente de *Nrwl* ha pensado en ello y el **Nx** instala y configura dos productos de última generación que facilitan la tarea. Usaremos [Jest](https://jestjs.io/) para los test unitarios y [Cypress](https://www.cypress.io/) para los de integración _End to End_. Vamos a empezar por el final.

Con cada aplicación generada se crea una hermana para sus pruebas _e2e_. Esa aplicación de pruebas está configurada y lista para compilar, servir y probar su aplicación objetivo. El comando `yarn e2e` lanzará el equivalente del cli `ng e2e`, el cual usará la configuración del `angular.json` para ejecutar **Cypress** con la configuración apropiada.

Tu trabajo como _tester_ será definir las pruebas en la carpeta `/integration`. Por ejemplo para empezar nos ofrecen el fichero `app-spec.ts` en el que yo he especificado el comportamiento deseado por mi página.


`integration/app.spec.ts`

```typescript
import { getAppLink, getImage } from '../support/app.po';

describe('GIVEN: an Angular SPA', () => {
  beforeEach(() => cy.visit('/'));
  context('WHEN: user visits home page', () => {
    it('THEN: should display link with app name', () => {
      getAppLink().contains('spa');
    });
    it('THEN: should display the logo', () => {
      getImage()
        .should('have.attr', 'src')
        .should('include', 'logo.png');
    });
  });
});
```

En la carpeta `/support` nos sugieren que creemos utilidades para tratar con el _DOM_ y de esa forma mantener los test lo más cercanos posible a un lenguaje natural de negocio. En mi caso una aproximación libre al [BDD con gherkin](https://www.genbeta.com/desarrollo/bdd-cucumber-y-gherkin-desarrollo-dirigido-por-comportamiento) para mantener el espíritu de sencillez de un tutorial sobre tecnología Angular, no sobre testing.

`support/app.po.ts`

```typescript
export const getAppLink = () => cy.get('nav > span > a');
export const getImage = () => cy.get('img');
```

---


# 8. e2e: Dada una página web en Angular.

> Dada una página web en Angular cuando se visita la página de inicio entonces debería mostrar un mensaje de bienvenida y luego debería mostrar el logotipo

Repito el proceso para la otra aplicación. Imagino que de esta forma verás las similitudes y sacarás conclusiones para optimizar tus pruebas. Puedes crear _scripts_ específicos para lanzar las pruebas de cada aplicación por separado, o dejar que **NX** ejecute **Cypress** para todo el repositorio.


`integration/app.spec.ts`

```typescript
import { getGreeting, getImage } from '../support/app.po';

describe('GIVEN: an Angular web', () => {
  beforeEach(() => cy.visit('/'));
  context('WHEN: user visits home page', () => {
    it('should display welcome message', () => {
      getGreeting().contains('Welcome to the Angular.Builders/blueprint for web!');
    });
    it('should display the logo', () => {
      getImage()
        .should('have.attr', 'src')
        .should('include', 'logo.png');
    });
  });
});
```

`support/app.po.ts`

```typescript
export const getGreeting = () => cy.get('h1');
export const getImage = () => cy.get('img');
```

---

# 9. e2e: Dada una librería en Angular con componentes de diseño.

> Dada una librería en Angular con componentes de diseño cuando visita una aplicación web que la consuma, entonces debería contener el componente header

En este caso queremos probar una librería de componentes. Claro que se podrán hacer pruebas unitarias y de integración parcial. Pero también puedes incluirlas como parte de la prueba de integración total de la aplicación que la consume. De esta forma te aseguras de que el módulo de la librería se importa y que sus componentes se exportan correctamente. Por ejemplo lo uso desde la aplicación web simple para comprobar que se renderiza el componente de cabecera `ab-layout-header`.


`integration/app.spec.ts`

```typescript
import { getAbLayoutHeader } from '../support/app.po';

describe('GIVEN: an Angular Library with layout components', () => {
  beforeEach(() => cy.visit('/'));
  context('WHEN: user visits a consumer app', () => {
    it('should contains an ab-layout-header element', () => {
      getAbLayoutHeader().should('exist');
    });
  });
});
```

`support/app.po.ts`

```typescript
export const getAbLayoutHeader = () =>
  cy.get( 'body > ab-web-root > ab-layout-header' );
```

---

# 10. unit: Dada una librería TypeScript con servicios.

> Dada una biblioteca de TypeScript con servicios cuando necesito un ConsoleLogger entonces debería ser creado
> Dado un registrador de trazas por consola cuando un dev quiere escribir un mensaje entonces debería ser capaz de dejar rastros amigables para el desarrollo.

Le toca el turno ahora a los tests unitarios. Estos los haremos con el sencillo y potente framework de pruebas **Jest**. En este caso también viene configurado por **Nx** y listo para ejecutar mediante el script `yarn test`. La sintaxis es similar a cualquier otra herramienta del mundillo y creo que cualquiera puede hacerse una idea viendo el siguiente ejemplo.

`services/console-tracer.driver.spec.ts`

```typescript
import { ConsoleTracerDriver } from './console-tracer.driver';

describe('Given an TypeScript Library with services', () => {
  describe('When I need a ConsoleTracerDriver', () => {
    it('then should be created', () => {
      const tracer: ConsoleTracerDriver = new ConsoleTracerDriver();
      expect(tracer).toBeTruthy();
    });
  });
});

describe('Given a Console Tracer Driver', () => {
  let tracer: ConsoleTracerDriver;
  beforeEach(() => {
    tracer = new ConsoleTracerDriver();
  });
  describe('When a devs wants to write a message', () => {
    it('then should be able to write dev friendly traces', () => {
      const consoleMessage = tracer.writeTrace({
        origin: 'test',
        level: 'system',
        message: 'test works'
      });
      expect(consoleMessage).toBe('[TEST]: test works');
    });
  });
});
```

---

# 11. unit: Dada una librería Angular con servicios de instrumentación.

> Dada una biblioteca Angular con servicios de instrumentación cuando la biblioteca se compila entonces debe crearse el módulo AngularTracerModule
> Dada una biblioteca Angular con servicios de instrumentación cuando necesito un servicio ConsoleTracer entonces debería ser creado

Y por supuesto que podemos hacer pruebas unitarias sobre aplicaciones o librerías **Angular**. Al menos probaremos con un _smoke test_ tanto la creación del módulo como la del servicio.

`tracer.module.spec.ts`

```typescript
import { async, TestBed } from '@angular/core/testing';
import { TracerModule } from './tracer.module';

describe('Given an Angular Library with instrumentation services ', () => {
  describe('When library compiles', () => {
    beforeEach(async(() => {
      TestBed.configureTestingModule({
        imports: [TracerModule]
      }).compileComponents();
    }));

    it('Then AngularTracerModule should be created ', () => {
      expect(TracerModule).toBeDefined();
    });
  });
});
```

`tracer.service.spec.ts`

```typescript
import { TestBed } from '@angular/core/testing';
import { TracerService, TRACER_CONFIG } from './tracer.service';

describe('Given an Angular Library with instrumentation services', () => {
  beforeEach(() => TestBed.configureTestingModule({}));

  describe('When I need a ConsoleTracer service', () => {
    it('Then should be created', () => {
      const service: TracerService = TestBed.get(TracerService);
      expect(service).toBeTruthy();
    });
  });
});
```

---

Para más información, o indicaciones paso a paso, consulta directamente la [documentación](https://angularbuilders.github.io/angular-blueprint/0-mono_repo) del proyecto en GitHub.

---

En definitiva, los grandes desarrollos demandados por bancos, multinacionales o administración pública requieren soluciones avanzadas. **Angular** es una plataforma ideal para esos grandes proyectos, pero requiere conocimiento y bases sólidas para sacarle partido.

Con este tutorial empiezas tu formación [avanzada en Angular](../tag/Avanzado/) para poder afrontar retos de tamaño industrial.

La iniciativa [Angular.Builders](https://angular.builders) nace para ayudar a desarrolladores y arquitectos de software como tú. Ofrecemos formación y productos de ayuda y ejemplo como [angular.blueprint](https://angularbuilders.github.io/angular-blueprint/).

Para más información sobre servicios de consultoría [ponte en contacto conmigo](https://www.linkedin.com/in/albertobasalo/).

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>
