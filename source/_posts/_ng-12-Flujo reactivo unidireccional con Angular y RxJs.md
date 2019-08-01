---
title: Flujo reactivo unidireccional con Angular y RxJs
permalink: flujo-reactivo-unidireccional-con-Angular-y-RxJs
date: 2019-08-01 19:46:34
tags:
- Angular
- Angular8
- RxJs
- Tutorial
- Avanzado
categories:
- [Tutorial, Angular]
thumbnail: /css/images/angular-12_unidirectional.png
---

![flujo-reactivo-unidireccional-con-Angular-y-RxJs](/images/tutorial-angular-12_unidirectional.png)

Continuando con el **tutorial de Angular Avanzado** nos centramos ahora en una arquitectura de comunicación de datos conocida como _Unidirectional Data Flow_ o flujo de datos en un mismo sentido. Esta técnica es una mejora sobre el modelo básico de Angular, el _double-binding_, que facilitaba mucho el desarrollo en pequeños proyectos.

> Cuando hablamos de mejora debemos ser honestos con los costes y beneficios: aquí vamos a mejorar la eficiencia en ejecución y a facilitar la depuración a costa de una mayor complejidad estructural y sintáctica. Merece la pena cuanto más grande sea el proyecto. Este es un ejemplo simplificado pero realista. Tómate tu tiempo para estudiarlo con calma.

Tomar las decisiones correctas en cuestiones de este calibre puede suponer la diferencia entre el éxito o fracaso de un proyecto. Voy a explicarte las razones para usar este patrón y la manera más sencilla de introducirlo en tus aplicaciones, dejándote en el umbral de soluciones aún más potentes como la gestión de estado centralizada con Redux.

<!-- more -->

Partiendo de la aplicación tal cómo quedó en [Nx, mono repositorios en Angular](../nx-mono-repositorios-en-Angular/). Al finalizar tendrás una aplicación que hace uso de un flujo unidireccional de datos y que funcionalmente proporciona un sistema de avisos sobre el uso de cookies en cumplimiento de las [política RGPD](https://es.wikipedia.org/wiki/Reglamento_General_de_Protecci%C3%B3n_de_Datos)

> Código asociado a este tutorial en _GitHub_: [angular.builders/angular-blueprint/](https://github.com/angularbuilders/angular-blueprint)
>
> > Tienes más información sobre este proyecto en [Angular.Builders](https://angular.builders/)

## Funcionalidad esperada:

Cuando un usuario visita la aplicación por primera vez debe avisarse del uso de cookies para cumplir con las políticas legales.

El aviso será mediante un dialogo flotante sobre la página.

El usuario puede ver más detalles o quedarse con la información reducida.

Según el tipo de aplicación, los detalles podrán verse en una página propia o en el mismo dialogo inicial.

El usuario podrá aceptar el uso de cookies.

Si lo hace, se almacenará la aceptación en el almacén local.

Habiendo aceptación no se mostrará más el dialogo.

## Tabla de Contenido:

[1. Un par de componentes contenedor/presentador con detección de cambio controlada.](./#1-Un-par-de-componentes-contenedor-presentador-con-deteccion-de-cambio-controlada)

[2. Todo reactivo.](./#2-Todo-reactivo)

[3. Un almacén genérico observable.](./#3-Un-almacen-generico-observable)

[4. Un servicio para mostrar u ocultar detalles](./#4-Un-servicio-para-mostrar-u-ocultar-detalles)

---

# 1. Un par de componentes contenedor/presentador con detección de cambio controlada.

> Como desarrollador quiero disponer de un componente para informar sobre RGPD a los usuarios de mis aplicaciones

Ya que este será un componente genérico y válido para muchas aplicaciones vamos a crearlo en una librería reutilizable a la que llamaré `policy`. En ella generamos los dos componentes. ¿Dos componentes?. Hemos dedicado un tema al [flujo de datos entre componentes](../lujo-de-datos-entre-componentes-angular) en el tutorial de introducción a Angular. Allí se recomendaba que la lógica de presentación y la de negocio estuviesen separadas.

Emergía el patrón _container-presenter_ que separa la responsabilidad de tratar con los servicios de datos de la responsabilidad de presentarlos en pantalla. Al componente que manipula datos se le llama contenedor, y al que los presenta... presentador. Puedes usar cualquier esquema de organización o nombrado. Yo los agrupo en dos carpetas: `containers` y simples `components`.

```bash
# Generate the policy library project
ng g library policy --tags=angular
# Generate the dialog components
# container
ng g c containers/mandatory-dialog --project=policy --export --inlineStyle --inlineTemplate --changeDetection=OnPush
# presenter
ng g c components/dialog --project=policy --changeDetection=OnPush
```

Eso sí todos llevan una modificador especial: el `changeDetection=OnPush`. Esta es la primera recomendación para cumplir con el flujo unidireccional. En esencia lo que hacemos es decirle a Angular que no use la detección de cambios por defecto. Tenemos un artículo con una explicación completa de la [Detección del cambio en Angular](../deteccion-del-cambio-en-Angular).


El componente contenedor suele tener poco _html_ y ningún _css_, así que es un candidato a entrar en un único fichero: `containers/mandatory-dialog.component.ts`. Nada interesante por el momento en este componente obligatorio en toda web pública.

```typescript
@Component({
  selector: 'ab-policy-mandatory-dialog',
  template: `
    <ab-policy-dialog>
    </ab-policy-dialog>
  `,
  styles: [],
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class MandatoryDialogComponent implements OnInit {
  constructor() {}
  public ngOnInit(): void {}
}
```

Tampoco el presentador requiere de mucha atención, así que muestro su simple vista: `components/dialog.component.html`

```html
<dialog open>
  <header>
    <p>This site uses cookies to personalize content, to provide social media features and to analyze traffic.</p>
  </header>
</dialog>
```

# 2. Todo reactivo.

> Como desarrollador quiero que las comunicaciones sean fluidas e independientes del tiempo para que los cambios en los datos cambien la presentación sin esfuerzo

Otro de los pilares de la programación moderna de grandes aplicaciones es la **reactividad** (nada que ver con Chernóbil). Se trata de que los cambios se comuniquen cuando ocurran, sin necesidad de preguntar por ellos. De esta forma **los componentes reaccionarán al cambio** en lugar de buscarlo proactivamente mejorando mucho el rendimiento de las aplicaciones.

Reducida a lo esencial, la lógica más básica que quiero implementar es un marcador que me indique si el usuario ha aceptado o no la política de cookies. Un mísero booleano. Pero claro, hacerlo reactivo requiere usar observables, y para eso emplearemos la librería [RxJs](https://www.learnrxjs.io/concepts/rxjs-primer.html).

Hablando de cosas esenciales. Las arquitecturas de software centradas en el dominio proponen que toda la lógica básica de una gran aplicación debería ser independiente de _pequeños detalles sin importancia como los frameworks_. Aprovechando las capacidades de [Nx para tratar con mono repos](../nx-mono-repositorios-en-Angular/) no cuesta nada crear un librería en dónde establecer en los modelos de datos y las entidades con sus reglas de negocio. A esta librería de dominio la llamaré `policy-domain`.

```bash
# Generate a policy-domain Type Script library with nx power-ups
ng g @nrwl/workspace:library policy-domain --directory=
```

Y en ella declaramos una clase que representa la entidad principal de este proyecto: la aceptación de las políticas. Esa clase informa a quien se suscriba de su estado de aceptación, inicialmente falso.

`libs\policy-domain\src\lib\services\policy-acceptation.entity.ts`

```typescript
export class PolicyAcceptationEntity {
  constructor() { }

  public isPolicyAccepted$(): Observable<boolean> {
    return of(false));
  }
}
```

En una capa superior, ya en un entorno Angular, haremos uso de la clase anterior en un servicio que a su vez expone un observable pero con su propia lógica adaptada de cara la vista.

```bash
# Generate a policy service
ng g s services/policy --project=policy
```

`libs\policy\src\lib\services\policy.service.ts`

```typescript
@Injectable({
  providedIn: 'root'
})
export class PolicyService {
  private policyAcceptationEntity = new PolicyAcceptationEntity();

  constructor( ) { }

  public haveToShowAccpetationDialog$(): Observable<boolean> {
    return this.policyAcceptationEntity
      .isPolicyAccepted$()
      .pipe(map(x => !x));
  }
}
```

Incorporamos el servicio en el componente principal para que nos indique si debemos mostrar el aviso al usuario o no.

`app.component.ts`

```typescript
export class AppComponent {
  public showPolicyDialog$: Observable<boolean>;

  constructor(private policyService: PolicyService) {
    this.showPolicyDialog$ = this.policyService.haveToShowAcceptationDialog$();
  }
}
```
Todo reactivo y todo asíncrono.

`app.component.html`

```html
<ab-policy-mandatory-dialog *ngIf="showPolicyDialog$ | async "></ab-policy-mandatory-dialog>
```

# 3. Un almacén genérico observable

> Como desarrollador quiero que los cambios en el estado estén controlados y desacoplados para saber quién lo cambia y quién se interesa

Hablar de _Unidirectional Data Flow_ sin presentar _Redux_ es poco menos que imposible. Y como tampoco es necesario liarse demasiado para sacarle partido vamos a verlo de manera práctica. Haciendo uso exclusivamente de la librería _RxJs_ y de las capacidades de _TypeScript_ vamos a crear un almacén, _store_ en la jerga _Redux_, observable.

De nuevo creamos una librería independiente de Angular que podamos usar en cualquier framework o incluso en _VanillaJS_. En esta librería pienso meter cosas estratégicas así que le pondré un nombre imponente: la llamaré `core-domain`

```bash
# Generate a core-domain library
ng g @nrwl/workspace:library core-domain --directory=
```

Y en ella una clase [genérica de TypeScript](https://www.typescriptlang.org/docs/handbook/generics.html) que tiene todos los atributos para cumplir con el principios _Redux_. Primero una intro teórica, luego el código y más adelante su consumo.

## 3.1 Principios de Redux

Tenemos tres principios básicos que cumplir:

- **Single Source Of Truth**: Cada pieza de información se almacena en un único lugar, independientemente de dónde sea creada, modificada o requerida.
- **Read Only State**: La información será de sólo lectura y sólo podrá modificarse a través de conductos oficiales.
- **Changes By Pure Functions**: Los cambios tienen que poder ser replicados, cancelados y auditados; lo mejor, usar una función pura.

## 3.2 Elementos de Redux

Los artificios fundamentales que incorporaremos a nuestro desarrollo van en dos niveles. El primer nivel resuelve los dos primeros principios.

- **Store**: El sistema que mantiene el estado. Despacha acciones de mutado sobre el mismo y comunica cambios enviando copias de sólo lectura a los subscriptores.
- **State**: Objeto que contiene la única copia válida de la información. Representa el valor del almacén en un momento determinado. Nunca expondremos un puntero a este dato privado.
- **Setters** : Métodos que asignan y notifican un nuevo cambio. Clonan la información recibida para que el llamante no tenga un puntero al estado..
- **Selectors** : Métodos para consulta del estado. Devuelven un observable al que suscribirse para obtener notificaciones de cambio o una instantánea. En cualquier caso siempre emitirá o devolverá un clon del estado.

Para cumplir con el tercer principio se necesita algo más. Al menos acciones y reductores. A partir de aquí ya no conviene reinventar la rueda y es mejor usar soluciones estándar.

- **Actions**: Objetos identificados por un tipo y cargados con un *payload*. Transmiten una intención de mutación sobre el estado del almacén. Tomados del patrón comando.
- **Reducers** : Son funciones puras, que ostentan la exclusividad de poder mutar el estado. Reciben dos argumentos: el estado actual y una acción con su tipo y su carga. Toman el estado, realizan los cambios oportunos y devuelven el estado mutado.
- **Effects** : Los reductores, como funciones puras, no pueden tener efectos secundarios. Es decir: depender o cambiar algo del entorno. Esto debería hacerse antes o después del cambio. En algo que aquí por ahora no usaremos: los efectos.


> Los funciones reductoras, al ser puras, mezclan la programación funcional con la orientada a objetos. Un reto pero también una demostración de la coexistencia de paradigmas en un mismo desarrollo. Me he tomado la libertad de simplificar un poco el uso de esta clase haciendo el reductor opcional y haciendo público el método `set(newState:T)` para casos dónde los cambios se quieran asignar por la vía rápida.

`libs\core-domain\src\lib\services\store.ts`

```typescript
import { BehaviorSubject, Observable } from 'rxjs';

export interface Action {
  type: string;
  payload: any;
}

export type Reducer<T> = (state: T, action: Action) => T;

function simpleDefaultReducer(state: any, action: Action): any {
  return action.payload;
}

export class Store<T> {
  private state$ = new BehaviorSubject<T>({ ...this.state });

  constructor(private state: T, private reducer: Reducer<T> = simpleDefaultReducer) {}

  public select(): T {
    return { ...this.state };
  }
  public select$(): Observable<T> {
    return this.state$.asObservable();
  }

  public dispatch(action: Action): void {
    const newState: T = this.reducer(this.state, action);
    this.set(newState);
  }
  public set(newState: T): void {
    this.state = { ...newState };
    this.state$.next(this.select());
  }
}
```

Esta es la implementación más sencilla posible de un almacén observable para empezar a trabajar con _Redux_.

# 4. Un servicio para mostrar u ocultar detalles

> Como usuario quiero ver y ocultar detalles sobre las cookies para saber qué se hace y decidir si acepto.

Empezamos con una interfaz para los datos del propietario del sitio y otra para controlar si se muestran o no los detalles. Después, el servicio propiamente dicho. El cual instancia una almacén observable en el que guardaremos el estado y al que notificaremos los cambios despachando acciones. Para simplificarlo no le paso ningún reductor y el sistema usará el que por defecto asigna la _payload_ sin más.

```typescript
export interface PolicyHolder {
  companyName: string;
  companyUrl: string;
  supportEmail: string;
  noticeUrl?: string;
}

export interface PolicyDetails {
  showingDetails: boolean;
  policyHolder: PolicyHolder;
}

export const POLICY_HOLDER_CONFIG = new InjectionToken<string>('policy-holder-config');

@Injectable({
  providedIn: 'root'
})
export class PolicyService {
  private policyAcceptationEntity: PolicyAcceptationEntity = new PolicyAcceptationEntity();
  private policyDetailsStore: Store<PolicyDetails>;

  constructor(@Inject(POLICY_HOLDER_CONFIG) public readonly policyHolderConfig: PolicyHolder) {
    this.policyDetailsStore = new Store<PolicyDetails>({
      showingDetails: false,
      policyHolder: policyHolderConfig
    });
  }

  public toggleMoreDetails() {
    const currentState = this.policyDetailsStore.select();
    currentState.showingDetails = !currentState.showingDetails;
    this.policyDetailsStore.dispatch({type: 'toggle', payload: currentState});
  }
```

# 5. Ya solo quedan los detalles

Permíteme el juego de palabras. Efectivamente queda mostrar un ocultar los detalles de la política de cookies. Con el pequeño de detalle de hacerlo en el propio dialogo o en una url aparte. Y queda el detalle de la  aceptación por parte del usuario y otro detalle más para un repositorio que lo almacene y recupere desde local storage. Me recuerda al chiste de cómo dibujar un caballo.

![Cómo dibujar un caballo](/images/draw-horse.jpeg)


Todos esos detalles con indicaciones paso a paso los tienes en la [documentación del Angular-Blueprint](https://angularbuilders.github.io/angular-blueprint/1-unidirectional) en GitHub y también se tratan en el [curso avanzado online](../https://academia-binaria.com/cursos/angular-business) que imparto con TrainingIT o a medida para tu empresa.

---

Las aplicaciones reales no son sencillas. Este es un tutorial avanzado que te exige conocimiento previo y dedicación. A cambio espero que te resulte útil y que podáis incorporar las técnicas de detección del cambio y control de estado observable en vuestros proyectos.

Con este tutorial continúas tu formación [avanzada en Angular](../tag/Avanzado/) para poder afrontar retos de tamaño industrial.

La iniciativa [Angular.Builders](https://angular.builders) nace para ayudar a desarrolladores y arquitectos de software como tú. Ofrecemos formación y productos de ayuda y ejemplo como [angular.blueprint](https://angularbuilders.github.io/angular-blueprint/).

Para más información sobre servicios de consultoría [ponte en contacto conmigo](https://www.linkedin.com/in/albertobasalo/).

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>
