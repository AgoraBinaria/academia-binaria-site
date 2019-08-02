---
title: Tests unitarios con Jest y e2e con Cypress en Angular
permalink: tests-unitarios-con-jest-y-e2e-con-cypress-en-Angular
date: 2019-07-18 11:25:12
tags:
- Angular
- Angular8
- Nx
- Test
- Jest
- Cypress
- Tutorial
- Avanzado
categories:
- [Tutorial, Angular]
thumbnail: /css/images/angular-11_test.png
---

![tests-unitarios-con-jest-y-e2e-con-cypress-en-Angular](/images/tutorial-angular-11_test.png)

Continuamos este **tutorial de Angular Avanzado** con el tema controvertido del testing. Sí, ya sé que todos deberíamos hacerlo siempre, pero también sé que no es cierto. Así que vamos a ponerle remedio eliminando excusas y facilitando las pruebas automatizadas.

> Por si hace falta su defensa: Las pruebas automáticas de código son la principal técnica de reducción de bugs y garantizan el buen funcionamiento durante un refactoring. Bueno para el usuario bueno para el programador.

Los desarrollos que hoy en día hacemos con **Angular** suelen ser de tamaño medio o grande y con una esperanza de vida y mantenimiento que se mide en años. Así que cuantas más pruebas tengamos menos miedo tendremos a cambiar el código. Y la necesidad de cambio siempre estará ahí. Veremos como _Jest_ y _Cypress_ nos ayudan muchísimo en la tarea.

De nuevo la gente de *Nrwl* ha pensado en ello y el **Nx** instala y configura dos productos de última generación que facilitan la tarea. Usaremos [Jest](https://jestjs.io/) para los test unitarios y [Cypress](https://www.cypress.io/) para los de integración _End to End_. Vamos a empezar por el final.

<!-- more -->

Partiendo del mono repositorio creado usando las herramientas de [Nrwl.io/](https://nrwl.io/) veremos que el **Nx** instala y configura dos productos de última generación que facilitan la tarea de las pruebas de software. Usaremos [Jest](https://jestjs.io/) para los test unitarios y [Cypress](https://www.cypress.io/) para los de integración _End to End_. Vamos a empezar por el final.

> Código asociado a este tutorial en _GitHub_: [angular.builders/angular-blueprint/](https://github.com/angularbuilders/angular-blueprint)


## Tabla de Contenido:

[1. e2e: Dada una SPA en Angular.](./#1-e2e-Dada-una-SPA-en-Angular)

[2. e2e: Dada una página web en Angular.](./#2-e2e-Dada-una-pagina-web-en-Angular)

[3. e2e: Dada una librería en Angular con componentes de diseño.](./#3-e2e-Dada-una-libreria-en-Angular-con-componentes-de-diseno)

[4. unit: Dada una librería TypeScript con servicios.](./#4-unit-Dada-una-libreria-TypeScript-con-servicios)

[5. unit: Dada una librería Angular con servicios de instrumentación.](./#5-unit-Dada-una-libreria-Angular-con-servicios-de-instrumentacion)

[Diagramas](./#Diagramas)

[Resumen](./#Resumen)

---

# 1. e2e: Dada una SPA en Angular.

> Dada una SPA en Angular cuando se visita la página de inicio entonces debería mostrar un enlace con el nombre de la aplicación y luego debería mostrar el logotipo

Con cada aplicación generada se crea una hermana para sus pruebas _e2e_. Esa aplicación de pruebas está configurada y lista para compilar, servir y probar su aplicación objetivo. El comando `yarn e2e` lanzará el equivalente del cli `ng e2e`, el cual usará la configuración del `angular.json` para ejecutar **Cypress** con la configuración apropiada.

> Cypress es el equivalente a Protractor, el producto propio de Angular. La ventaja principal es que no está atado a ningún framework y por tanto lo que hagas valdrá para probar cualquier web en cualquier tecnología.

Tu trabajo como _tester_ será definir las pruebas en la carpeta `/integration`. Por ejemplo para empezar nos ofrecen el fichero `app-spec.ts` en el que yo he especificado el comportamiento deseado por mi página. La sintaxis se enciende por si misma. He seguido el convenio GIVEN, WHEN, THEN para especificar pruebas que podrían considerarse casi de comportamiento o aceptación.

`integration/app.spec.ts`

```typescript
import { getAppLink, getImage } from '../support/app.po';

describe('GIVEN an Angular SPA', () => {
  beforeEach(() => cy.visit('/'));
  context('WHEN user visits home page', () => {
    it('THEN should display link with app name', () => {
      getAppLink().contains('spa');
    });
    it('THEN should display the logo', () => {
      getImage()
        .should('have.attr', 'src')
        .should('include', 'logo.png');
    });
  });
});
```

La parte más técnica y tediosa es la que accede al DOM y lo mejor es tener eso a parte. En la carpeta `/support` nos sugieren que creemos utilidades para tratar con el _DOM_ y de esa forma mantener los test lo más cercanos posible a un lenguaje natural de negocio. Como se ve en mi caso una aproximación libre al [BDD con gherkin](https://www.genbeta.com/desarrollo/bdd-cucumber-y-gherkin-desarrollo-dirigido-por-comportamiento) para mantener el espíritu de sencillez de un tutorial sobre tecnología Angular, no sobre testing.

`support/app.po.ts`

```typescript
export const getAppLink = () => cy.get('nav > span > a');
export const getImage = () => cy.get('img');
```

---


# 2. e2e: Dada una página web en Angular.

> Dada una página web en Angular cuando se visita la página de inicio entonces debería mostrar un mensaje de bienvenida y luego debería mostrar el logotipo

Repito el proceso para la otra aplicación. Imagino que de esta forma verás las similitudes y sacarás conclusiones para optimizar tus pruebas. Puedes crear _scripts_ específicos para lanzar las pruebas de cada aplicación por separado, o dejar que **NX** ejecute **Cypress** para todo el repositorio.


`integration/app.spec.ts`

```typescript
import { getGreeting, getImage } from '../support/app.po';

describe('GIVEN an Angular web', () => {
  beforeEach(() => cy.visit('/'));
  context('WHEN user visits home page', () => {
    it('THEN should display welcome message', () => {
      getGreeting().contains('Welcome to the Angular.Builders/blueprint for web!');
    });
    it('THEN should display the logo', () => {
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

# 3. e2e: Dada una librería en Angular con componentes de diseño.

> Dada una librería en Angular con componentes de diseño cuando visita una aplicación web que la consuma, entonces debería contener el componente header

En este caso queremos probar una librería de componentes. Claro que se podrán hacer pruebas unitarias y de integración parcial. Pero también puedes incluirlas como parte de la prueba de integración total de la aplicación que la consume. De esta forma te aseguras de que el módulo de la librería se importa y que sus componentes se exportan correctamente. Por ejemplo lo uso desde la aplicación web simple para comprobar que se renderiza el componente de cabecera `ab-layout-header`.


`integration/app.spec.ts`

```typescript
import { getAbLayoutHeader } from '../support/app.po';

describe('GIVEN an Angular Library with layout components', () => {
  beforeEach(() => cy.visit('/'));
  context('WHEN user visits a consumer app', () => {
    it('THEN should contains an ab-layout-header element', () => {
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

# 4. unit: Dada una librería TypeScript con servicios.

> Dada una biblioteca de TypeScript con servicios cuando necesito un ConsoleLogger entonces debería ser creado
> Dado un registrador de trazas por consola cuando un dev quiere escribir un mensaje entonces debería ser capaz de dejar rastros amigables para el desarrollo.

Le toca el turno ahora a los **tests unitarios**. Estos los haremos con el sencillo y potente framework de pruebas **Jest**. En este caso también viene configurado por **Nx** y listo para ejecutar mediante el script `yarn test`. La sintaxis es similar a cualquier otra herramienta del mundillo y creo que cualquiera puede hacerse una idea viendo el siguiente ejemplo.

`services/console-tracer.driver.spec.ts`

```typescript
import { ConsoleTracerDriver } from './console-tracer.driver';

describe('GIVEN an TypeScript Library with services', () => {
  describe('WHEN I need a ConsoleTracerDriver', () => {
    it('THEN should be created', () => {
      const tracer: ConsoleTracerDriver = new ConsoleTracerDriver();
      expect(tracer).toBeTruthy();
    });
  });
});

describe('GIVEN a Console Tracer Driver', () => {
  let tracer: ConsoleTracerDriver;
  beforeEach(() => {
    tracer = new ConsoleTracerDriver();
  });
  describe('WHEN a dev wants to write a message', () => {
    it('THEN should be able to write dev friendly traces', () => {
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

# 5. unit: Dada una librería Angular con servicios de instrumentación.

> Dada una biblioteca Angular con servicios de instrumentación cuando la biblioteca se compila entonces debe crearse el módulo AngularTracerModule
> Dada una biblioteca Angular con servicios de instrumentación cuando necesito un servicio ConsoleTracer entonces debería ser creado

Y por supuesto que podemos hacer pruebas unitarias sobre aplicaciones o librerías **Angular**. Al menos probaremos con un _smoke test_ tanto la creación del módulo como la del servicio.

`tracer.module.spec.ts`

```typescript
import { async, TestBed } from '@angular/core/testing';
import { TracerModule } from './tracer.module';

describe('GIVEN an Angular Library with instrumentation services ', () => {
  describe('WHEN the library compiles', () => {
    beforeEach(async(() => {
      TestBed.configureTestingModule({
        imports: [TracerModule]
      }).compileComponents();
    }));

    it('THEN the AngularTracerModule should be created ', () => {
      expect(TracerModule).toBeDefined();
    });
  });
});
```

`tracer.service.spec.ts`

```typescript
import { TestBed } from '@angular/core/testing';
import { TracerService, TRACER_CONFIG } from './tracer.service';

describe('GIVEN an Angular Library with instrumentation services', () => {
  beforeEach(() => TestBed.configureTestingModule({}));

  describe('WHEN I need a ConsoleTracer service', () => {
    it('THEN should be created', () => {
      const service: TracerService = TestBed.get(TracerService);
      expect(service).toBeTruthy();
    });
  });
});
```

---

## Diagramas

El siguiente diagrama nos muestra a vista de pájaro las aplicaciones de prueba _e2e_ y las aplicaciones reales.

![Dependencias entre proyectos](/images/11-projects-dependency.png)

---

Para más información, o indicaciones paso a paso, consulta directamente la [documentación](https://angularbuilders.github.io/angular-blueprint/1-test) del proyecto en GitHub.

Las tareas relativas a este tutorial resueltas en el [proyecto 1 - test](https://github.com/angularbuilders/angular-blueprint/projects/4)

![Angular.Builders](/css/images/angular.builders.png)

La iniciativa [Angular.Builders](https://angular.builders) nace para ayudar a desarrolladores y arquitectos de software como tú. Ofrecemos formación y productos de ayuda y ejemplo como [angular.blueprint](https://angularbuilders.github.io/angular-blueprint/).

Para más información sobre servicios de consultoría [ponte en contacto conmigo](https://www.linkedin.com/in/albertobasalo/).

---

## Resumen

En definitiva, los grandes desarrollos demandados por bancos, multinacionales o administración pública requieren soluciones fiables y mantenibles. Y eso pasa inexcusablemente por hacer testing. **Angular** facilita las pruebas unitarias y de integración; especialmente con las herramientas _Jest_ y  _Cypress_ ya configuradas por **Nx**.

Con este tutorial de formación [avanzada en Angular](../tag/Avanzado/) te preparas para poder afrontar retos de tamaño industrial. Continúa aprendiendo a mejorar el rendimiento usando la [detección del cambio en Angular](../deteccion-del-cambio-en-Angular).


> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>
