---
title: Detección del cambio en Angular
permalink: deteccion-del-cambio-en-Angular
date: 2019-08-01 18:09:27
tags:
- Angular
- Angular8
- ChangeDetection
- Tutorial
- Avanzado
categories:
- [Tutorial, Angular]
thumbnail: /css/images/angular-12_change.png
---

![deteccion-del-cambio-en-Angular](/images/tutorial-angular-12_change.png)

La forma en que Angular realiza su renderizado y actualiza sus vistas es un factor clave para el rendimiento de las aplicaciones. ¿Cómo funciona la estrategia de detección de cambios de Angular? Pues tiene dos modos: `default` o *automágico* y `onPush` o *mindfulness*.

> Es importante tomar consciencia sobre el proceso y las implicaciones. Es costoso realizar la detección más veces de lo necesario y porque no hacerlo suficientemente implica no ver resultados reales.

Afortunadamente el cambio del modo automático al manual no tiene por qué ser traumático. Con un mayor conocimiento del `changeDetectionStrategy` y un poco de trabajo extra tendrás aplicaciones más eficientes y mantenibles.


<!-- more -->

Partiendo de la aplicación tal cómo quedó en [Tests unitarios con Jest y e2e con Cypress en Angular](../tests-unitarios-con-jest-y-e2e-con-cypress-en-Angular/). Al finalizar tendrás una aplicación que actualiza la vista sólo cuando es necesario, es decir: cuando los datos han cambiado.

> Código asociado a este tutorial en _GitHub_: [angular.builders/angular-blueprint/](https://github.com/angularbuilders/angular-blueprint)


## Tabla de Contenido:

[1. Comunicación de datos entre componentes.](./#1-Comunicacion-de-datos-entre-componentes)

[2. Change detection strategies](./#2-Change-detection-strategies)

[3. Inmutabilidad](./#3-Inmutabilidad)

[Ejemplo](./ejemplo)

[4. Un par de componentes con detección de cambio controlada.](./#4-Un-par-de-componentes-con-deteccion-de-cambio-controlada)

[5. Todo reactivo.](./#5-Todo-reactivo)

[Diagramas](./diagramas)

[Resumen](./resumen)

---

# 1 Comunicación de datos entre componentes

> Como desarrollador quiero disponer de un componente para informar sobre RGPD a los usuarios de mis aplicaciones

La detección de cambios se dispara ante eventos que le ocurren a los componentes. **La detección se realiza componente a componente**, así que compensa tener muchos componentes pequeños, para que cada uno por si sólo no genere demasiado ruido.

## 1.1 Componentes Contenedores y Presentadores

Al pasar de un único componente a varios mini-componentes, se propone usar **el patrón contenedor / presentadores**. Se mantiene un componente padre que contiene múltiples componentes presentadores hijos. El contendor es el único con acceso a los servicios de negocio y datos. Los presentadores reciben los datos y emiten eventos. Los presentadores no obtienen ni modifican datos por su cuenta.

Nomenclatura
- **Container**: aka *Parent, Smart*. Irán en una subcarpeta `containers`
- **Presenter**: aka *Child, Dumb*. Irán en una subcarpeta... `components`

> Este reparto de responsabilidades es aconsejable independientemente de la estrategia de detección aplicada. Tienes más información el artículo de introducción [flujo de datos entre componentes](../lujo-de-datos-entre-componentes-angular).

# 2 Change detection strategies

Con la aplicación bien estructurada en componentes y con la comunicación estandarizada, habremos reducido el impacto de la detección del cambio y estaremos preparados para optimizarlo. Conozcamos en detalle las estrategias de detección del cambio.

El decorador `@Component()` admite en su configuración la poco conocida propiedad `changeDetection`. Dicha propiedad puede asignarse manualmente al componente, o indicarle su uso al generador del cli.

## 2.1 Detección automática, *default*

Por defecto, Angular tiene que ser conservador y verificar cada posible cambio, esto se denomina comprobación sucia o *dirty checking*. Se dispara **con demasiada frecuencia**, al menos en los siguientes casos:

- Eventos desde el browser
- Timers, intervals etc..
- Llamadas http
- Promesas y código asíncrono.

Por si fuera poco, además de dispararse mucho es **muy costoso**. Determinar que algo ha cambiado implica comparar dos estados: el actual y el anterior.

> La comparación es valor a valor, en profundidad, para cada propiedad de cada objeto, para cada objeto de cada array.

Con todo, esta estrategia es cómoda para el programador y suficiente para casos básicos. Pero demasiada magia dificulta el control en aplicaciones complejas. Y en pantallas de mucha información e interacción degrada el rendimiento percibido.

## 2.2 Detección manual, *onPush*

Como se puede prever, la detección del cambio manual es lanzada por el programador. No siempre va a ser laborioso, pero será más consciente pues para que ocurra han de darse alguna de estas circunstancias:

- **Explícitamente** el programador solicita la detección llamando a `ChangeDetectorRef.detectChanges();
- **Implícitamente** al usar el `pipe Async` en la vista se llama a ese mismo método.
- **Conscientemente** el desarrollador obliga a un componente a repintarse si le cambia la referencia a un `@Input()`.

> En este modo los componentes dejan de evaluar y comparar sus propiedades rutinariamente. Sólo atienden a eventos `@Output()` o **cambios de referencia** `@Input()`. Esto relaja mucho al motor de Angular, que ya no tiene que hacer comparaciones odiosas. Sabrá que algo ha cambiado porque... es otro objeto.

# 3 Inmutabilidad

Como ya se ha dicho, para que Angular en la estrategia automática decida que algo ha cambiado necesita hacer una comparación por valor. Para evitar ese coste usamos la estrategia manual y el programador tiene que cambiar la referencia de algo cuando quiera que Angular repinte la vista.

## 3.1 Por referencia y por valor

Normalmente tendrá que crear un nuevo objeto y reasignarlo en lugar del anterior en un **ciclo de clonación, mutación y asignación**. Por costoso que parezca siempre compensa si evita muchas e innecesarias comparaciones por valor en estructuras profundas.

La estrategia `onPush` trata a todos los `Inputs` en inmutables, es decir, algo que no espera que cambie. Similar al paso de parámetros por valor, que si cambia es porque es otro puntero.

## 3.2 El clonado
El potencialmente pesado trabajo de clonado lo podemos evitar en muchos casos usando alguna de estas técnicas:

- **Tipos primitivos** que se pasan por valor en las propiedades `@Input()`
- **Arrays**: muchos métodos como `.filter() .slice() .sort() .concat()` etc., devuelven nuevas referencias sin modificar el array original.
- **Observables y el pipe Async**, pues en este caso se subscribe y lanza implícitamente la detección del cambio. Sin necesidad de clonar.

Para los demás casos tenemos operadores *TypeScript* sencillos y optimizados para obtener nuevas referencias a partir de otros ya existentes.

```typescript
const original = { name:'first', value:1 };
const cloned = { ...original };
// > { name:'first', value:1 }
const mutated = { ...original, value:2, newProperty: 'added' };
// > { name:'first', value:2, newProperty: 'added' }
const list = [ original, cloned, mutated ];
// > [ { name:'first', value:1 }, { name:'first', value:1 }, { name:'first', value:2, newProperty: 'added' } ]
const clonedList = [ ...list ];
// > [ { name:'first', value:1 }, { name:'first', value:1 }, { name:'first', value:2, newProperty: 'added' } ]
const mutatedList = [ ...list, { name: 'new item'} ];
// > [ { name:'first', value:1 }, { name:'first', value:1 }, { name:'first', value:2, newProperty: 'added' }, { name: 'new item'} ]
const newList = list.filter(i => i.name=='first');
// > [ { name:'first', value:1 }, { name:'first', value:1 }, { name:'first', value:2, newProperty: 'added' } ]
```

Ya tienes los conocimientos para acelerar y reducir la incertidumbre sobre el actualización de vistas usando el patrón **contenedor / presentador** junto con la estrategia de detección de cambios `OnPush`. Ahora vamos a ver un ejemplo.

## Ejemplo

### Funcionalidad esperada:

Cuando un usuario visita la aplicación por primera vez debe avisarse del uso de cookies para cumplir con las políticas legales.

El aviso será mediante un dialogo flotante sobre la página.

El usuario puede ver más detalles o quedarse con la información reducida.

# 4. Un par de componentes con detección de cambio controlada.

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

# 5. Todo reactivo.

> Como desarrollador quiero que las comunicaciones sean fluidas e independientes del tiempo para que los cambios en los datos cambien la presentación sin esfuerzo

Otro de los pilares de la programación moderna de grandes aplicaciones es la **reactividad** (nada que ver con Chernóbil). Se trata de que los cambios se comuniquen cuando ocurran, sin necesidad de preguntar por ellos. De esta forma **los componentes reaccionarán al cambio** en lugar de buscarlo proactivamente mejorando mucho el rendimiento de las aplicaciones.

Reducida a lo esencial, la lógica más básica que quiero implementar es un marcador que me indique si el usuario ha aceptado o no la política de cookies. Un mísero booleano. Pero claro, hacerlo reactivo requiere usar observables, y para eso emplearemos la librería [RxJs](https://www.learnrxjs.io/concepts/rxjs-primer.html).

Hablando de cosas esenciales. Las arquitecturas de software centradas en el dominio proponen que toda la lógica básica de una gran aplicación debería ser independiente de _pequeños detalles sin importancia como los frameworks_. Aprovechando las capacidades de [Nx para tratar con mono repos](../nx-mono-repositorios-en-Angular/) no cuesta nada crear un librería en dónde establecer en los modelos de datos y las entidades con sus reglas de negocio. A esta librería de dominio la llamaré `policy-domain`.

```bash
# Generate a policy-domain Type Script library with nx power-ups
ng g @nrwl/workspace:library policy-domain --directory=
```

Y en ella declaramos una clase que representa la entidad principal de este proyecto: la aceptación de las políticas. Esa entidad es una clase informa a quien se suscriba de su estado de aceptación, inicialmente falso.

`libs\policy-domain\src\lib\services\policy-acceptation.entity.ts`

```typescript
export class PolicyAcceptationEntity {
  constructor() { }

  public isPolicyAccepted$(): Observable<boolean> {
    return of(false));
  }
}
```

En una capa superior, ya en un entorno Angular, haremos uso de la entidad de dominio anterior. Será un servicio que a su vez va a exponer un observable, pero con su propia lógica adaptada, de cara la vista.

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

Al utilizar observables, podemos usar el _pipe_ `async`, que ya sabemos que informa de los cambios al detector. Por tanto la detección `OnPush` es perfectamente válida.

---

## Diagramas

El siguiente diagrama nos muestra a vista de pájaro las librerías y aplicaciones implicadas hasta el momento.

![Dependencias entre proyectos](/images/12-projects-dependency.png)

![Dependencias entre componentes y clases](/images/12-class-dependency.png)

---

Para más información, o indicaciones paso a paso, consulta directamente la [documentación](https://angularbuilders.github.io/angular-blueprint/2-change) del proyecto en GitHub.

Las tareas relativas a este tutorial resueltas en el [proyecto 2 - change-detection](https://github.com/angularbuilders/angular-blueprint/projects/2)

![Angular.Builders](/css/images/angular.builders.png)

La iniciativa [Angular.Builders](https://angular.builders) nace para ayudar a desarrolladores y arquitectos de software como tú. Ofrecemos formación y productos de ayuda y ejemplo como [angular.blueprint](https://angularbuilders.github.io/angular-blueprint/).

Para más información sobre servicios de consultoría [ponte en contacto conmigo](https://www.linkedin.com/in/albertobasalo/).

---

## Resumen

La detección automática es cómoda pero costosa. Por dos razones: se dispara muchas veces y necesita comprobar si hay cambios comparando valor por valor. La detección manual es más eficiente. Se lanza menos veces y además le basta un cambio de referencia para saber que hay novedades. Para poder usarla sin grandes trabajos recomiendo usar el _pipe_ `async`, siempre con orígenes de datos observables.

Para ello necesitamos usar y conocer patrones que hagan uso de la librería observable _RxJs_. En este tutorial de formación [avanzada en Angular](../tag/Avanzado/) te muestra como mejorar el rendimiento usando el [Flujo reactivo unidireccional con Angular y RxJs](../flujo-reactivo-unidireccional-con-Angular-y-RxJs).

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>
