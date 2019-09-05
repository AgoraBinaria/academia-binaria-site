---
title: Tests unitarios con Jest y e2e con Cypress en Angular
permalink: tests-unitarios-con-jest-y-e2e-con-cypress-en-Angular
date: 2019-09-05 14:25:12
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

<!-- more -->

Partiendo del [mono repositorio creado](../nx-mono-repositorios-en-Angular) usando las herramientas de [Nrwl.io/](https://nrwl.io/) aprovecharemos las herramientas que instala y configura dos productos de última generación que facilitan la tarea. Usaremos [Jest](https://jestjs.io/) para los test unitarios y [Cypress](https://www.cypress.io/) para los de integración _End to End_. Vamos a empezar por el final.

> Código asociado a este tutorial en _GitHub_: [AcademiaBinaria/angular-boss](https://github.com/AcademiaBinaria/angular-boss)


## Tabla de Contenido:

[1. Test de Integración con Cypress.](./#1.-Test-de-Integración-con-Cypress)

[2. Test Unitarios con Jest.](./#2.-Test-Unitarios-con-Jest)

<!-- [Diagramas](./#Diagramas) -->

[Resumen](./#Resumen)

---

# 1. Test de Integración con Cypress

```yaml
As a: developer,
  I want: to test end to end my app
  so that: I can be sure of the functions
```

Las pruebas de integración son las que tienen un retorno de inversión más inmediato. Si esta es tu primera aproximación al testing, te recomiendo empezar por aquí. Se trata de definir un escenario y pre programar el comportamiento del usuario. El sistema lo ejecutará y podrás contrastar el resultado con algún valor esperado.

Son pruebas de caja negra que interactúan con el sistema en ejecución. Todo ello automatizado y con informes visuales interactivos o en ficheros de texto con el resultado de las pruebas.

## 1.1 Cypress

[Cypress](https://www.cypress.io/) es el equivalente a Protractor, el producto propio de Angular. La ventaja principal es que no está atado a ningún framework y por tanto lo que hagas valdrá para probar cualquier web en cualquier tecnología.

Con cada aplicación generada se crea una hermana para sus pruebas _e2e_. Esa aplicación de pruebas está configurada y lista para compilar, servir y probar su aplicación objetivo. Con Nx tenemos todo instalado, configurado y listo para ejecutarse con comandos como los siguientes que yo coloco en el `packcage.json`:

```json
  "e2e:shop": "ng e2e shop-e2e --watch",
  "e2e:warehouse": "ng e2e warehouse-e2e --watch",
```

Luego se pueden lanzar desde la terminal muy cómodamente.

```terminal
yarn e2e:shop
yarn e2e:warehouse
```

## 1.2 Test e2e

```yaml
GIVEN: the shop web app
  WHEN: user visits home page
    THEN: should display welcome message
    THEN: should display welcome message from the API
```

Tu trabajo como _tester_ será definir las pruebas en la carpeta `/integration`. Por ejemplo para empezar nos ofrecen el fichero `app-spec.ts` en el que yo he especificado el comportamiento deseado por mi página. La sintaxis se enciende por si misma. He seguido el convenio **GIVEN, WHEN, THEN** para especificar pruebas que podrían considerarse casi de comportamiento o aceptación.

`apps\shop-e2e\src\integration\app.spec.ts`

```typescript
import { getGreeting } from '../support/app.po';

describe('GIVEN: the shop web app', () => {
  beforeEach(() => cy.visit('/'));
  context('WHEN: user visits home page', () => {
    it('THEN: should display welcome message', () => {
      getGreeting().contains('Hello world');
    });
    // needs the api server to run
    // yarn start:api
    it('THEN: should display welcome message from the API', () => {
      getGreeting().contains('and Welcome to api!');
    });
  });
});
```

La parte más técnica y tediosa es la que accede al DOM y lo mejor es tener eso a parte. En la carpeta `/support` nos sugieren que creemos utilidades para tratar con el _DOM_ y de esa forma mantener los test lo más cercanos posible a un lenguaje natural de negocio. Como se ve en mi caso una aproximación libre al [BDD con gherkin](https://www.genbeta.com/desarrollo/bdd-cucumber-y-gherkin-desarrollo-dirigido-por-comportamiento) para mantener el espíritu de sencillez de un tutorial sobre tecnología Angular, no sobre testing.

`apps\shop-e2e\src\support\app.po.ts`

```typescript
export const getGreeting = () => cy.get('h1');
```

---


# 2. Test Unitarios con Jest

Las pruebas unitarias, muy asociadas al TDD, son las mejores amigas del desarrollador. Pero, también son las más engorrosas para empezar. así que aquí veremos una introducción sencilla para que nadie deje de hacerlas.


## 2.1 Jest

[Jest](https://jestjs.io/) es un framework de testing para JavaScript muy sencillo y rápido. Puedes usarlo con cualquier otro framework. Para el caso de angular ya viene preconfigurado si usas las extensiones Nx.

Mete los siguientes scripts en el package.json y así tendrás a mano siempre las pruebas. Te recomiendo que desarrolles con el test unitario lanzado, es la manera más rápida de probar el código que estés tocando. Idealmente incluso con el test antes del código.

```json
  "test:shop": "ng test shop --watch --verbose",
  "test:warehouse": "ng test warehouse --watch --verbose",
  "test:api": "ng test api --watch --verbose",
```

```terminal
yarn test:shop
yarn test:warehouse
yarn test:api
```

## 2.2 Tests unitarios


### 2.2.1 Componentes

```yaml
GIVEN: an AppComponent declared in AppModule
  WHEN: the AppModule is compiled
    THEN: should create the component
    THEN: should have a property title with value 'shop'
    THEN: should render 'Hello world' in a H1 tag
```

En este caso queremos probar una librería de componentes. Claro que se podrán hacer pruebas unitarias y de integración parcial. Pero también puedes incluirlas como parte de la prueba de integración total de la aplicación que la consume. De esta forma te aseguras de que el módulo de la librería se importa y que sus componentes se exportan correctamente. Por ejemplo lo uso desde la aplicación shop puedo comprobar su componente `AppComponent` y que se renderiza también el componente con los saludos `ab-ui-greetings`.


`apps\shop\src\app\app.component.spec.ts`

```typescript
import { UiModule } from '@a-boss/ui';
import { async, TestBed } from '@angular/core/testing';
import { RouterTestingModule } from '@angular/router/testing';
import { AppComponent } from './app.component';

describe('GIVEN: an AppComponent declared in AppModule', () => {
  describe('WHEN: the AppModule is compiled', () => {
    beforeEach(async(() => {
      TestBed.configureTestingModule({
        imports: [RouterTestingModule, UiModule],
        declarations: [AppComponent]
      }).compileComponents();
    }));

    it('THEN: should create the component', () => {
      const fixture = TestBed.createComponent(AppComponent);
      const app = fixture.debugElement.componentInstance;
      expect(app).toBeTruthy();
    });

    it(`THEN: should have a property title with value 'shop'`, () => {
      const fixture = TestBed.createComponent(AppComponent);
      const app = fixture.debugElement.componentInstance;
      expect(app.title).toEqual('shop');
    });

    it(`THEN: should render 'Hello world' in a H1 tag`, () => {
      const fixture = TestBed.createComponent(AppComponent);
      fixture.detectChanges();
      const compiled = fixture.debugElement.nativeElement;
      expect(compiled.querySelector('h1').textContent).toContain('Hello world');
    });
  });
});
```

### 2.2.2 Services

```yaml
GIVEN: a GreetingsService
  WHEN: the DataModule is compiled
    THEN: should be created
    THEN: should return an observable when call 'getGrettings()'
    THEN: should return 'Welcome to api!' when call 'getGrettings()'
```

`libs\shared\data\src\lib\greetings\greetings.service.spec.ts`

---

```typescript
import {
  HttpClientTestingModule,
  HttpTestingController
} from '@angular/common/http/testing';
import { async, TestBed } from '@angular/core/testing';
import { Observable } from 'rxjs';
import { GreetingsService } from './greetings.service';

describe('GIVEN: a GreetingsService', () => {
  describe('WHEN: the DataModule is compiled', () => {
    beforeEach(() => {
      TestBed.configureTestingModule({
        imports: [HttpClientTestingModule]
      });
    });

    it('THEN: should be created', () => {
      const service: GreetingsService = TestBed.get(GreetingsService);
      expect(service).toBeTruthy();
    });

    it(`THEN: should return an observable when call 'getGrettings()'`, () => {
      const service: GreetingsService = TestBed.get(GreetingsService);
      const greetings$: Observable<any> = service.getGrettings$();
      expect(greetings$).toBeInstanceOf(Observable);
    });

    // Ojo al async para ejectuar las llamadas asíncronas
    it(`THEN: should return 'Welcome to api!' when call 'getGrettings()'`, async(() => {
      const service: GreetingsService = TestBed.get(GreetingsService);
      service
        .getGrettings$()
        .subscribe(result =>
          expect(result).toEqual({ message: 'Welcome to api!' })
        );
      const httpMock = TestBed.get(HttpTestingController); // mock del backend para no depender del servidor
      const req = httpMock.expectOne('http://localhost:3333/api'); // esperar a que se llame a esta ruta
      req.flush({ message: 'Welcome to api!' }); // responder con esto
      httpMock.verify(); // comprobar que no hay más llmadas
    }));
  });
});
```

---

## Resumen

En definitiva, los grandes desarrollos demandados por bancos, multinacionales o administración pública requieren soluciones fiables y mantenibles. Y eso pasa inexcusablemente por hacer testing. **Angular** facilita las pruebas unitarias y de integración; especialmente con las herramientas _Jest_ y  _Cypress_ ya configuradas por **Nx**.

Con este tutorial de formación [avanzada en Angular](../tag/Avanzado/) te preparas para poder afrontar retos de tamaño industrial. Continúa aprendiendo a mejorar el rendimiento usando la [detección del cambio en Angular](../deteccion-del-cambio-en-Angular).


> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>
