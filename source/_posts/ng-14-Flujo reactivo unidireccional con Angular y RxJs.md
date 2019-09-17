---
title: Redux, flujo reactivo unidireccional con Angular y RxJs
permalink: flujo-reactivo-unidireccional-con-Angular-y-RxJs
date: 2019-08-05 11:46:34
tags:
- Angular
- Angular8
- Redux
- RxJs
- Tutorial
- Avanzado
categories:
- [Tutorial, Angular]
thumbnail: /css/images/angular-14_redux.png
---

![flujo-reactivo-unidireccional-con-Angular-y-RxJs](/images/tutorial-angular-14_redux.png)

Continuando con el **tutorial de Angular Avanzado** nos centramos ahora en una arquitectura de comunicación de datos conocida como _Unidirectional Data Flow_ o flujo de datos en un mismo sentido. Esta técnica es una mejora sobre el modelo básico de Angular, el _double-binding_, que facilitaba mucho el desarrollo en pequeños proyectos.

> Cuando hablamos de mejora debemos ser honestos con los costes y beneficios: aquí vamos a mejorar la eficiencia en ejecución y a facilitar la depuración a costa de una mayor complejidad estructural y sintáctica. Merece la pena cuanto más grande sea el proyecto. Este es un ejemplo simplificado pero realista. Tómate tu tiempo para estudiarlo con calma.

Tomar las decisiones correctas en cuestiones de este calibre puede suponer la diferencia entre el éxito o fracaso de un proyecto. Voy a explicarte las razones para usar este patrón y la manera más sencilla de introducirlo en tus aplicaciones, dejándote en el umbral de soluciones aún más potentes como la gestión de estado centralizada con Redux.

<!-- more -->

Partiendo de la aplicación tal cómo quedó en [componentes dinámicos, directivas y pipes con Angular](../componentes-dinamicos-directivas-y-pipes-con-Angular). Al finalizar dotaremos a la aplicación de un sistema de avisos sobre el uso de cookies en cumplimiento de la [política RGPD](https://es.wikipedia.org/wiki/Reglamento_General_de_Protecci%C3%B3n_de_Datos)

> Código asociado a este tutorial en _GitHub_: [angular.builders/angular-blueprint/](https://github.com/angularbuilders/angular-blueprint)


## Tabla de Contenido:

[Ejemplo](./#Ejemplo)

[1. Un almacén genérico observable.](./#1-Un-almacen-generico-observable)

[2. Un servicio para mostrar u ocultar detalles.](./#2-Un-servicio-para-mostrar-u-ocultar-detalles)

[3. Ya solo quedan los detalles.](./#3-Ya-solo-quedan-los-detalles)

[Diagramas](./#Diagramas)

[Resumen](./#Resumen)


---
## Ejemplo

### Funcionalidad esperada:

- Cuando un usuario visita la aplicación por primera vez debe avisarse del uso de cookies para cumplir con las políticas legales.

- El aviso será mediante un diálogo flotante sobre la página.

- El usuario puede ver más detalles o quedarse con la información reducida.

- Según el tipo de aplicación, los detalles podrán verse en una página propia o en el mismo diálogo inicial.

- El usuario podrá aceptar el uso de cookies.

- Si lo hace, se almacenará la aceptación en el almacén local.

- Habiendo aceptación no se mostrará más el diálogo.


# 1. Un almacén genérico observable

> Como desarrollador quiero que los cambios en el estado estén controlados y desacoplados para saber quién lo cambia y quién se interesa

Hablar de _Unidirectional Data Flow_ sin presentar _Redux_ es poco menos que imposible. Y como tampoco es necesario liarse demasiado para sacarle partido vamos a verlo de manera práctica. Haciendo uso exclusivamente de la librería _RxJs_ y de las capacidades de _TypeScript_ vamos a crear un almacén (un _store_ en la jerga _Redux_) observable.

De nuevo creamos una librería independiente de Angular que podamos usar en cualquier _framework_ o incluso en _VanillaJS_. En esta librería pienso meter cosas estratégicas así que le pondré un nombre imponente: la llamaré `core-domain`

```bash
# Generate a core-domain library
ng g @nrwl/workspace:library core-domain --directory=
```

Y en ella una clase [genérica de TypeScript](https://www.typescriptlang.org/docs/handbook/generics.html) que tiene todos los atributos para cumplir con el principios _Redux_. Primero una intro teórica, luego el código y más adelante su consumo.

## 1.1 Principios de Redux

Tenemos tres principios básicos que cumplir:

- **Single Source Of Truth**: Cada pieza de información se almacena en un único lugar, independientemente de dónde sea creada, modificada o requerida.
- **Read Only State**: La información será de sólo lectura y sólo podrá modificarse a través de conductos oficiales.
- **Changes By Pure Functions**: Los cambios tienen que poder ser replicados, cancelados y auditados; lo mejor, usar una función pura.

## 1.2 Elementos de Redux

Los artificios fundamentales que incorporaremos a nuestro desarrollo van en dos niveles. El primer nivel resuelve los dos primeros principios.

- **Store**: El sistema que mantiene el estado. Despacha acciones de mutado sobre el mismo y comunica cambios enviando copias de sólo lectura a los subscriptores.
- **State**: Objeto que contiene la única copia válida de la información. Representa el valor del almacén en un momento determinado. Nunca expondremos un puntero a este dato privado.
- **Setters** : Métodos que asignan y notifican un nuevo cambio. Clonan la información recibida para que el llamante no tenga un puntero al estado.
- **Selectors** : Métodos para consulta del estado. Devuelven un observable al que suscribirse para obtener notificaciones de cambio o una instantánea. En cualquier caso siempre emitirá o devolverá un clon del estado.

Para cumplir con el tercer principio se necesita algo más. Al menos acciones y reductores. A partir de aquí ya no conviene reinventar la rueda y es mejor usar soluciones estándar. En próximos artículos te mostraré como hacerlo con _NgRx_. usando _Actions_,  _Reducers_ y _Effects_. Por ahora nos quedamos con lo básico un estado privado con capacidad de notificar cambios.


`libs\core-domain\src\lib\services\store.ts`

```typescript
import { BehaviorSubject, Observable } from 'rxjs';

export class Store<T> {
  private state$ = new BehaviorSubject<T>({ ...this.state });

  constructor(private state: T) {}

  public select(): T {
    return { ...this.state };
  }
  public select$(): Observable<T> {
    return this.state$.asObservable();
  }

  public set(newState: T): void {
    this.state = { ...newState };
    this.state$.next(this.select());
  }
}
```

Esta es la implementación más sencilla posible de un almacén observable para empezar a trabajar con _Redux_. Usamos un `BehaviorSubject` para notificar cambios, aunque sólo exponemos su interfaz `asObservable()` . Por lo demás lo único obligatorio es usar clones `{ ...this.state }` tanto al recibir como al devolver el valor del estado.

# 2. Un servicio para mostrar u ocultar detalles

> Como usuario quiero ver y ocultar detalles sobre las cookies para saber qué se hace y decidir si acepto.

Empezamos con una interfaz para los datos del propietario del sitio y otra para controlar si se muestran o no los detalles y después el servicio propiamente dicho. El servicio tienen instancia del almacén observable con el tipo concreto `PolicyDetails`. Será el lugar en el que guardaremos el estado y al que notificaremos los cambios. Para simplificarlo no uso por ahora ningún reductor y se asignará el nuevo valor sin más mediante el método set.

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
    this.policyDetailsStore.set(currentState);
  }

  public targetUrlToNavigateForDetails$(): Observable<string> {
    return this.policyDetailsStore.select$().pipe(
      filter(x => x.policyHolder.noticeUrl !== undefined),
      map(x => (x.showingDetails ? x.policyHolder.noticeUrl : ''))
    );
  }
```

Como ves el servicio no expone directamente nada que tenga que ver con el `store`. Sus métodos público son exclusivamente de negocio. Asigna cambio y filtra y transforma las notificaciones para ser consumidas directamente.

# 3. Ya solo quedan los detalles

Permíteme el juego de palabras. Efectivamente queda mostrar u ocultar los detalles de la política de cookies. Con el pequeño detalle de hacerlo en el propio diálogo o en una url aparte. Y queda también el detalle de la  aceptación por parte del usuario; y claro, falta otro detalle más para tener un repositorio que lo almacene y recupere desde _local storage_. Me recuerda al chiste de cómo dibujar un caballo.

![Cómo dibujar un caballo](/images/draw-horse.jpeg)

Al menos si que quiero mostrarte cómo se consume el servicio desde el componente principal de la aplicación. Se suscribe a los cambios que le indican que debe navegar a otra url, pero no sabe qué los provocó.

```typescript
export class AppComponent {
  public title = 'spa';
  public showPolicyDialog$: Observable<boolean>;

  constructor(private router: Router, private policyService: PolicyService) {
    this.showPolicyDialog$ = this.policyService.haveToShowAcceptationDialog$();
    this.policyService
      .targetUrlToNavigateForDetails$()
      .subscribe({ next: this.navigateTo.bind(this) });
  }
  private navigateTo(targetUrl: string) {
    this.router.navigate([targetUrl]);
  }
}
```
Está completamente desacoplado del componente en el que el usuario indicó que quería ver más detalles. Sólo comparte la instancia `PolicyService` y se comunican mediante comandos y suscripción a cambios.

```typescript
@Component({
  selector: 'ab-policy-mandatory-dialog',
  template: `
    <ab-policy-dialog
      *ngIf="(policyDetails$ | async) as policyDetails"
      [policyDetails]="policyDetails"
      (toggleDetails)="onToggleDetails()"
    >
    </ab-policy-dialog>
  `
})
export class MandatoryDialogComponent {
  public policyDetails$: Observable<PolicyDetails> = this.policyService.policyDetails$();

  constructor(private policyService: PolicyService) {}

  public onToggleDetails(): void {
    this.policyService.toggleMoreDetails();
  }
}
```

## Diagramas

En los siguientes diagramas se muestra a vista de pájaro las librerías y clases implicadas hasta el momento para no perderte mientras sigues todo el código necesario.

![Dependencias entre proyectos](/images/13-projects-dependency.png)

![Dependencias entre componentes y clases](/images/13-class-dependency.png)

---

Todos esos detalles con indicaciones paso a paso los tienes en la [documentación del Angular-Blueprint](https://angularbuilders.github.io/angular-blueprint/3-unidirectional) en GitHub y también se tratan en el [curso avanzado online](../https://academia-binaria.com/cursos/angular-business) que imparto con TrainingIT o a medida para tu empresa.

Las tareas relativas a este tutorial resueltas en el [proyecto 2 - change-detection](https://github.com/angularbuilders/angular-blueprint/projects/2)

![Angular.Builders](/css/images/angular.builders.png)

La iniciativa [Angular.Builders](https://angular.builders) nace para ayudar a desarrolladores y arquitectos de software como tú. Ofrecemos formación y productos de ayuda y ejemplo como [angular.blueprint](https://angularbuilders.github.io/angular-blueprint/).

Para más información sobre servicios de consultoría [ponte en contacto conmigo](https://www.linkedin.com/in/albertobasalo/).

---

## Resumen

Las aplicaciones reales no son sencillas. Este es un tutorial avanzado que te exige conocimiento previo y dedicación. A cambio espero que te resulte útil y que podáis incorporar las técnicas de detección del cambio y control de estado observable en vuestros proyectos.

Con este tutorial continúas tu formación [avanzada en Angular](../tag/Avanzado/) para poder afrontar retos de tamaño industrial.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>
