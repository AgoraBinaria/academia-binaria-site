---
title: El patrón Redux con NgRx en Angular
permalink: el-patron-redux-con-ngrx-en-angular
date: 2018-10-08 12:50:27
tags:
- Angular
- Angular8
- Angular2
- Redux
- NgRx
- Tutorial
- Avanzado
categories:
- [Tutorial, Angular]
thumbnail: /css/images/angular-a_redux.png
---

![el-patron-redux-con-ngrx-en-angular](/images/tutorial-angular-a_redux.png)

Le pasa a todas las aplicaciones. Crecen y crecen en funcionalidad y complejidad. En Angular estamos preparados para modularizar, componentizar e inyectar servicios. Pero con grandes aplicaciones, o con grandes equipos, parece que nada es suficiente. Se necesita una **gestión del estado centralizada** como la del patrón Redux.

**Redux no hace rápido lo simple, sino mantenible lo complejo**. Y si tienes delante un desarrollo complejo, te recomiendo que uses *NgRX*; la solución estándar para implementar **Redux con Angular**.


<!-- more -->

Partiendo del código tal cómo quedó en [Detección del cambio en Angular](../el-patron-redux-con-ngrx-en-angular/). Al finalizar tendrás una aplicación que gestiona centralizadamente los cambios, que permite conocer qué ocurrió y predecir lo que ocurrirá usando NgRx.

> Código asociado a este tutorial en _GitHub_: [AcademiaBinaria/angular-boss](https://github.com/AcademiaBinaria/angular-boss)


https://academiabinaria.github.io/angular-boss/5-ngrx.html#1

# 1 El patrón Redux

Redux es como una base de datos, es en un almacén para el estado de la aplicación. Pero un almacén que gestiona sus cambios de manera predictiva. Combina dos patrones: *Action* para el envío de comandos para actualizar el estado del almacén; y *Observable* para la subscripción a cambios en el estado del almacén. **Desacopla los emisores de acciones de los receptores de cambios** en los datos.

Para lograrlo tendremos que cumplir una serie de principios y utilizar unos elementos predefinidos. Esto introduce capas de abstracción y burocracia que inicialmente complican el desarrollo. Pero a medio plazo harán que mantener la aplicación sea mucho más sencillo y seguro.

## 1.1 Principios de Redux

Tenemos tres principios básicos que cumplir:

- **Single Source Of Truth**: Cada pieza de información se almacena en un único lugar, independientemente de dónde sea creada, modificada o requerida.
- **Read Only State**: La información será de sólo lectura y sólo podrá modificarse a través de conductos oficiales.
- **Changes By Pure Functions**: Los cambios tienen que poder ser replicados, cancelados y auditados; lo mejor, usar una función pura.

## 1.2 Elementos de Redux

Estos son los artificios fundamentales que incorporaremos a nuestro desarrollo:

- **Store**: El sistema que mantiene el estado. Despacha acciones de mutado sobre el mismo y comunica cambios enviando copias de sólo lectura a los subscriptores.
- **State**: Árbol de objetos que contienen la única copia válida de la información. Representa el valor del almacén en un momento determinado.
- **Actions**: Objetos identificados por un tipo y cargados con un *payload*. Transmiten una intención de mutación sobre el estado del *store*.
- **Reducers** : Son funciones puras, que ostentan la exclusividad de poder mutar el estado. Reciben dos argumentos: el estado actual y una acción con su tipo y su carga. Clonan el estado, realizan los cambios oportunos y devuelven el estado mutado.


- **Actions**: Objetos identificados por un tipo y cargados con un *payload*. Transmiten una intención de mutación sobre el estado del almacén. Tomados del patrón comando.
- **Reducers** : Son funciones puras, que ostentan la exclusividad de poder mutar el estado. Reciben dos argumentos: el estado actual y una acción con su tipo y su carga. Toman el estado, realizan los cambios oportunos y devuelven el estado mutado.
- **Effects** : Los reductores, como funciones puras, no pueden tener efectos secundarios. Es decir: depender o cambiar algo del entorno. Esto debería hacerse antes o después del cambio. En algo que aquí por ahora no usaremos: los efectos.

> Los funciones reductoras, al ser puras, mezclan la programación funcional con la orientada a objetos. Un reto pero también una demostración de la coexistencia de paradigmas en un mismo desarrollo.


# 2 NgRx

NgRx es el estándar de facto para implementar Redux en Angular. Está basada en *RxJS* y es una librería modular con todo lo necesario para crear grandes aplicaciones. Esto son los módulos que la componen:

- **store**: Es el módulo principal con el administrador del estado centralizado y reactivo.
- **store-devtools**: Instrumentación para depurar desde el navegador. Vale su peso en oro.
- **router-Store** : Almacena el estado del *router* de Angular en el *store*, tratando cada evento como una acción Redux.
- **effects**: Los reductores son funciones puras sin efectos colaterales. Este módulo es la solución para comandos asíncronos.
- **schematics, entity, ngrx-data**: Son otros módulos opcionales con ayudas y plantillas de NgRX.

## 2.1 Instalación y configuración

Para agregar *NgRx* a un app te propongo la siguiente receta de comandos. Además de esto tendrás que instalar en tu navegador las herramientas de diagnóstico [Redux DevTools](http://extension.remotedev.io/).

```shell
npm i @angular-devkit/schematics --save-dev
npm i @ngrx/schematics --save-dev
ng config cli.defaultCollection @ngrx/schematics
npm i @ngrx/store --save
npm i @ngrx/store-devtools --save
ng g st RootState --root -m app.module.ts --spec false
npm install @ngrx/router-store --save
```
Con esto habrás instalado y configurado NgRx en tu app. Completa tu módulo principal para que tenga algo así:

```typescript
@NgModule({
  imports: [
    CommonModule,
    RouterModule,
    StoreModule.forRoot(rootReducers, { metaReducers }),
    StoreRouterConnectingModule.forRoot({ stateKey: 'router' }),
    !environment.production ? StoreDevtoolsModule.instrument() : []
  ]
})
export class AppModule {}
```

> El código generado por `@ngrx/schematics` no es para todos los gustos. Tómalo como un punto de partida y crea la estructura que mejor te encaje. En [Autobot](https://github.com/AcademiaBinaria/autobot/tree/a-redux/src/app/core/store/state) he empezado por llevarlo todo al `CoreModule` y guardarlo en una carpeta propia convenientemente nombrada `store/state`. Veámosla en detalle.

## 2.2 State y reducers, los cambios de estado mediante reductores

El estado en *redux* es un objeto tipado a partir de una interfaz, normalmente llamada `State` a secas, aunque yo prefiero identificarla como `RootState`. Tendrá propiedades para almacenar objetos más o menos complejos. Cada propiedad tendrá su propio tipo complejo y será gestionada por una función reductora específica. Se necesita un proceso de mapeo que asigne la función reductora a cada propiedad del estado raíz. Eso es lo que hace este código:

```typescript
export interface RootState {
  router: any;
  global: GlobalState;
  cars: CarsState;
}

export const rootReducers: ActionReducerMap<RootState> = {
  router: routerReducer,
  global: globalreducer,
  cars: carsReducer
};
```

Las funciones reductoras pueden estar en cualquier fichero. Son puras y no deben ser incluidas en ninguna clase. Normalmente tendrás un fichero para cada reductor. El del *router* ya viene hecho por NgRx, todos los demás son cosa tuya. Por ejemplo este es el reductor sobre la propiedad `global: GlobalState`.

```typescript
export function globalReducer(state = initialState, action: GlobalActions): GlobalState {
  switch (action.type) {
    case GlobalActionTypes.SendUserMessage:
      return { ...state, userMessage: action.payload };
    case GlobalActionTypes.IsLoginNeeded:
      return { ...state, loginNeeded: action.payload };
    case GlobalActionTypes.StoreToken:
      return { ...state, token: action.payload };
    default:
      return state;
  }
}
```

Como puedes ver esta función recibe dos argumento: el estado y la acción que pretende mutarlo. **La acción es un objeto** con dos propiedades. La primera y obligatoria es el tipo y sirve para escoger la lógica que se aplicará. La otra, opcional, es el `payload` que actúa como argumento de la acción. El reductor no puede usar nada más que estos argumentos: **state y action**. Y por ser puro no puede mutar ninguno, por tanto tiene que clonar el estado recibido antes de aplicarle cualquier cambio. Una vez efectuado el cambio, usando el *payload* si es preciso, devolverá el clon mutado.

## 2.3 Actions, las acciones permitidas

El patrón Redux no obliga a grandes cosas respecto a cómo implementar las acciones. Lo único que de verdad necesitas es crear objetos con una propiedad obligatoria `type` y otra opcional `payoload`.

Pero en NgRX han decidido aprovechar al máximo las capacidades del *TypeScript* y proponen usar un código fuertemente tipado. Para empezar crean un `enum` que detalla los posible tipos de una acción y les asigna un texto para que luzcan y faciliten su búsqueda en *logs*.

```typescript
export enum GlobalActionTypes {
  SendUserMessage = '[Global] Show Message',
  IsLoginNeeded = '[Auth] Is Login Needed',
  StoreToken = '[Auth] Store Token'
}
```

Y claro, en un ambiente tipado, la acción será una clase que ha de cumplir una interfaz. En dicha clase queda ya predeterminado su tipo de acción; y por supuesto que permiten fijar el tipo de datos de la `payload`. Todo ello genera un código como el siguiente.

```typescript
export class SendUserMesage implements Action {
  readonly type = GlobalActionTypes.SendUserMessage;
  constructor(public payload: string) {}
}
```

Para terminar, y ya que habremos de crear un buen número de clases para las acciones, también nos propone exprimir *TypeScript* para crear un tipo combinado que ofrezca a los consumidores el plantel completo y delimitado de acciones posibles.

```typescript
export type GlobalActions = SendUserMesage | IsLoginNeeded | StoreToken;
```

Es normal que al principio todo este *boilerplate* te parezca un exceso. Pero entre las plantillas, los *snippets* y el *copy and paste* no es tan trabajoso como pudiera parecer. A cambio tienes un código robusto y explícitamente detallado.

## 2.4 Dispatch y Select, despacho de acciones y selección de cambios

Desde el resto del código **la comunicación con este estado centralizado se realizará por dos conductos oficiales : Dispatch y Select**. El primero es un método que recibe como único parámetro una instancia de una acción. Dicha instancia lleva implícito el tipo y puede ir rellena de una *payload*. Así es como se envían los cambios hacia el almacén.

```typescript
constructor(private store: Store<RootState>) {
  this.store.dispatch(new SendUserMesage('Tutorial Angular en Español'));
}
```

No esperamos respuesta de este método. Es un mundo de *fire and forget*. Pero en cualquier otro lugar o lugares de nuestro código podremos recibir notificaciones de cualquier cambio que se haya producido. De forma desacoplada podremos recibir el mensaje con una simple subscripción *RxJS*.

```typescript
this.store
  .select('global', 'userMessage')
  .subscribe(userMessage: string => console.log(userMessage));
```

A este método de consulta asíncrona se le envía un argumento con múltiples sobrecargas llamado *selector*. En este caso está formado por dos claves que apuntan a una propiedad concreta en la que estoy interesado: `global.userMessage`.

Con esto tenemos una primera implementación del patrón Redux en Angular usando NgRx. Pero aún hay más.

# 3 Efectos y módulos funcionales

Las funciones reductoras, como ya se ha dicho, deben ser puras. La idea es que puedan ser auditadas, re-ejecutadas y testeadas sin que necesiten servicios externos ni causen efectos colaterales. Y esto es un problema con la cantidad de **ejecuciones asíncronas en las aplicaciones web**. Cualquier tentación de lanzar una llamada *AJAX* dentro de un reductor debe ser elimindad de inmediato.

> Dos razones: por un lado en Angular se necesita invocar al *httpClient* de alguna manera para realizar la llamada *AJAX*. Y, ya que la función no pertenece a ninguna clase, no puede haber constructor que reclame la inyección de dicho servicio. Tampoco las funciones puras tienen permitido usar nada que no venga entre sus argumentos. Por otra parte las funciones puras han de ser predecibles, y una llamada a un servidor remoto no es en absoluto predecible. Puede pasarle de todo, así que los reductores no son país para procesos asíncronos.

La solución que proponen *NgRX* es usar un artificio llamado efecto, porque será encargado de **los efectos secundarios que provocan las las instrucciones asíncronas**. De una forma simplista, diremos que las acciones asíncronas se multiplicarán por tres. El comando que genera la llamada, y los dos potenciales eventos con la respuesta correcta o el error.

Para manejarlo todo incluyen en la librería el módulo `EffectsModule` que ha de registrarse junto al `StoreModule`. Desde ese momento *NgRX* activa un sistema de seguimiento que trata las acciones como un stream de *RxJS* y permite subscribirse a la invocación de dichas acciones y tratarlas adecuadamente. Una vez más, aprovechan las características del *TypeScript* y hacen uso de los decoradores para definir las funciones que responderán a la ejecución de las acciones.

## 3.1 @Effect(), efectos secundarios

El decorador `@Effect()` se aplica a propiedades de servicios estándar de Angular. *NgRX* invoca esas funciones ante cada acción despachada, así que lo primero que debemos hacer es aplicar un filtro para quedarnos con las acciones del tipo que nos interese. El que lanzará la llamada `http`.

> Todo el trabajo se realizará con *streams* y requiere de un conocimiento previo de la librería *RxJS* y de la mecánica de su método `pipe` y sus operadores reactivos.

Cuando llegue una de esas acciones realizaremos la llamada asíncrona sin complejos. Recordemos que un efecto forma parte de un servicio inyectable de Angular. No tienen ninguna restricción funcional por parte de Redux. Obviamente la respuesta debe ser capturada y tratada según haya sido correcta o no.

Si recibimos una respuesta válida podremos retornar directamente una nueva acción que actualice el estado con los datos recibidos. En cambio si ha habido un error, habrá que reactivar el *stream* con un nuevo observable que emita la acción que procesará el error.

```typescript
@Effect()
constructor(private actions$: Actions, private cars: CarsService) {}

public loadCarsEffect$: Observable<CarsActions> = this.actions$.pipe(
  ofType<LoadCars>(CarsActionTypes.LoadCars),
  mergeMap(() =>
    this.cars.getCars$().pipe(
      map((cars: Car[]) => new LoadCarsOK(cars)),
      catchError(err => of(new LoadCarsError('Error loading cars')))
    )
  )
);
```

Tómate el tiempo necesario para comprender este código. Especialmente los operadores `ofType` de *NgRX* y el `mergeMap` de *RxJS*. Fíjate bien en los tipos de los datos recibidos y devueltos. El efecto trata con observables de acciones. Recibe una y devuelve otra que puede ser de alguno de los otros dos tipos previstos. En el caso correcto se cambia una acción por otra, en otro caso se crea un nuevo observable mediante el constructor `of()` antes del retorno.

> Para un ejemplo más completo de estos conceptos explora la implementación en el proyecto Autobot. El [RootStore](https://github.com/AcademiaBinaria/autobot/tree/a-redux/src/app/core) hace uso de efectos en la carpeta `store/state/cars`.

## 3.2 Feature, módulos funcionales

Ya que Redux está especialmente indicado en grandes aplicaciones pero manteniendo un estado centralizado, es fácil que acabemos con un módulo raíz demasiado pesado que ralentice el inicio de la aplicación. Para adaptar la gestión central a un entorno con módulos cargado con  *lazyloading* necesitamos una última ayuda, de la mano de `FeatureModule`.

Esencialmente es una nuevo *store* supeditado al principal pero que no se define, y por tanto no pesa, hasta que no es necesario. En Autobot el [CarModule](https://github.com/AcademiaBinaria/autobot/tree/a-redux/src/app/car) es un buen ejemplo de *store* cargado de forma dinámica como una `Feature`.

Ya tienes los conocimientos para gestionar de manera centralizada, auditable y predecible el estado de tus programas. El patrón Redux lucirá más cuanto más grande y compleja sea tu aplicación.

Continúa tu formación avanzada para crear aplicaciones [PWA, entre la web y las apps con Angular](../pwa-entre-la-web-y-las-apps-con-angular) y verás como aprendes a programar con Angular 8.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>
