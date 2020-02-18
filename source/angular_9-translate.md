> Ayudita conceptual...
>> https://www.campusmvp.es/recursos/post/angular-9-ya-esta-aqui-y-estas-son-sus-novedades.aspx

# Un vistazo a las características principales en el lanzamiento de Angular Ivy versión 9

[editor de publicaciones](https://admin.indepth.dev/ghost/#/editor/post/5e2f4cae430b1e0c3681c749)

[Lars Gyrup Brink Nielsen](https://indepth.dev/author/layzee/) 06 de febrero de 2020 9 min de lectura

MUCHO y en todas partes, globalización dinámica, modo estricto, Bazel y mucho más.

* (Sí, ahora se ha lanzado Angular Ivy versión 9) *

## Ivy está habilitado por defecto

En versiones anteriores de Angular, tuvimos que optar por Ivy. En la versión 9, tenemos que optar por no participar en Ivy si queremos recurrir a View Engine. Esto es posible en ambas versiones 9 y 10 para garantizar una transición más fluida de View Engine a Ivy.

Las librerías *se pueden* compilar AOT, pero no es recomendable. El equipo de Angular tiene un plan de migración de View Engine a Ivy que recomienda publicar solo librerías compatibles con View Engine compiladas con JIT para Angular versión 9. El compilador de compatibilidad Angular actualizará las librerías compatibles con View Engine a Ivy cuando se instalen en una aplicación Angular Ivy.

[Obtén información sobre la compatibilidad de la biblioteca y el plan de transición de View Engine-to-Ivy en "La guía de Angular Ivy para autores de librerías"](https://indepth.dev/the-angular-ivy-guide-for-library-authors/) .

```json
// tsconfig.json
{
  "angularCompilerOptions": {
    "enableIvy": false
  }
}
```

```typescript
// polyfills.ts
// Solo se usa en aplicaciones multilingües de Ivy
// import '@angular/localize/init';
```

*Listado 1. Optar por no usar Ivy para recurrir a View Engine.*

Si tienes problemas con Ivy en tu aplicación, o en cualquiera de las librerías de las que depende, puedes optar por salir de Ivy y recurrir a View Engine. Hazlo desactivando la opción del compilador angular `enableIvy` y deshabilitando` @angular/localize` como se ve en Listado 1.

Optar por evitar Ivy en un entorno de servidor es un poco más complicado. [Sigue la guía oficial para darte de baja de Ivy cuando uses la renderización del lado del servidor](https://angular.io/guide/ivy#using-ssr-without-ivy).

## El principio de localidad

Antes, para compilar un componente en View Engine, Angular necesitaba información sobre todas sus dependencias declaradas. Esto significa que las librerías Angular no se pueden compilar AOT usando View Engine.

Ahora para compilar un componente en Ivy, Angular solo necesita información sobre el componente en sí. Pero, Ivy no necesita metadatos de ninguna dependencia declarable para compilar un componente.

El principio de localidad significa que, en general, veremos tiempos de construcción más rápidos.

### Componentes con carga diferida

Las declaraciones `entryComponents` están en desuso porque ya no son necesarias. Cualquier componente Ivy puede cargarse de forma diferida y renderizarse dinámicamente.

Esto significa que ahora podemos cargar de forma diferida y renderizar un componente sin enrutamiento o módulos Angular. Sin embargo, en la práctica aún tenemos que usar módulos para renderizar componentes o módulos funcionales para vincular la plantilla de un componente a sus dependencias.

Las librerías que solo son utilizadas por un componente con carga diferida se agrupan en fragmentos con carga diferida.

## Mejoras en la carga diferencial

Cuando se introdujo la carga diferencial en Angular versión 8, el proceso de compilación se ejecutaba una vez para el paquete ES5 y otra vez para el paquete ES2015+.

En Angular versión 9, primero se emite un paquete ES2015+. Ese paquete se transfiere a un paquete ES5 separado. De esta manera, no tenemos que pasar por un proceso de compilación completo dos veces.

## Compilación AOT en todas partes

AOT está habilitado de forma predeterminada en las compilaciones, el servidor de desarrollo e incluso en las pruebas. Anteriormente, la compilación AOT era significativamente más lenta que la compilación JIT, por lo que JIT se usaba para el desarrollo y las pruebas. Con las mejoras de tiempo de construcción y reconstrucción en Ivy, la compilación AOT ahora ofrece una gran experiencia al desarrollador.

Cuando utilizábamos la compilación JIT en desarrollo y solo la compilación AOT en la compilación final, algunos errores se detectaban solo al hacer compilaciones de producción o, peor aún, en tiempo de ejecución.

## Tamaños de paquete

Ivy puede habilitar paquetes más pequeños porque usa el _Ivy Instruction Set_, que es un conjunto de instrucciones de representación de tiempo de ejecución potencialmente **tree-shakable**. Nuestros paquetes solo incluirán las instrucciones de renderizado que realmente usamos en nuestros proyectos.

Esto es ideal para casos de uso como _microfrontends_, Angular Elements y aplicaciones web donde Angular no controla todo el documento.

Sin embargo, la diferencia en los tamaños de nuestros paquetes entre View Engine e Ivy variará según el tamaño de nuestra aplicación y las librerías de terceros que utilizamos. En general:

- Las aplicaciones pequeñas y simples verán una disminución considerable del peso del paquete.
- Las aplicaciones complejas verán un aumento en el fichero principal, pero una disminución en los tamaños de paquetes con carga diferida.

Esto significa una disminución considerable del tamaño de descarga combinada para aplicaciones grandes, pero podría significar un aumento en el peso total para aplicaciones medianas. En ambos casos, el tamaño del paquete principal probablemente aumentará, lo que es malo para el tiempo de carga inicial de la página.

## Globalización

Las configuraciones regionales (formato de número, formato de fecha y otras configuraciones regionales) se pueden cargar dinámicamente en tiempo de ejecución en lugar de tener que registrarse en tiempo de compilación.

```typescript
// main.ts
import '@angular/localize/init';

import { loadTranslations } from '@angular/localize';

loadTranslations ({
  '8374172394781134519': '¡Hola, {$nombre de usuario}! Bienvenido a {$appName}. ',
});
```

*Listado 2. Carga dinámica de traducciones.*

Como se ve en el Listado 2, los textos traducidos también se pueden cargar dinámicamente en tiempo de ejecución en lugar de ser parte de nuestros paquetes.

Los textos traducidos pueden cargarse desde una base de datos o un archivo.

### Múltiples idiomas desde un único paquete de aplicaciones

Para cambiar el idioma, tenemos que reiniciar la aplicación, pero no tenemos que servir un fichero de aplicaciones diferente.

Esto significa que podemos, con alguna configuración, admitir varios idiomas con un solo paquete de aplicaciones en un solo nombre de host.

### Tiempo de compilación en línea

Una aplicación localizada ahora solo se compilará una vez. En lugar de múltiples compilaciones para producir un paquete por idioma, se produce un paquete por idioma reemplazando los marcadores de posición `$localize` con textos traducidos.

Ahora  hay que agregar el paquete `@angular/localize` para posibilitar la localización (varios idiomas). La buena noticia es que ya no tenemos que incluir el código de localización de Angular en nuestros paquetes si solo tenemos un idioma.

Si no usamos plantillas localizadas, las instrucciones de Ivy `i18n *` se sacan del paquete principal.

### Textos localizables en modelos de componentes y servicios

```typescript
// app.component.ts
@Component({
  template: '{{ title }}'
})
export class AppComponent {
  title = $localize`Welcome to MyApp`;
}
```

*Listado 3. Un marcador de posición de texto de traducción en un modelo de componente.*

Una nueva característica de internacionalización es que también podemos incluir marcadores de posición para textos traducidos en el contenido de nuestros modelos como se ve en el Listado 3. Anteriormente, esto solo era posible en las plantillas.

## Ámbitos adicionales de proveedores de dependencias

Siempre hemos tenido alcance de módulo para los proveedores de inyección de servicios. La versión Angular 6 introdujo el alcance `'root'` y los proveedores _tree-shakable_, tanto para los proveedores de alcance desde el módulo raíz como desde módulos inferiores.

La versión angular 9 presenta los nuevos ámbitos de proveedor `'platform'` y `'any'`. Los proveedores con ámbito de `'platform'` se pueden compartir entre múltiples aplicaciones Angular en el mismo documento. El alcance del proveedor `'any'` compartirá un proveedor por inyector de módulo. Por ejemplo, una instancia de servicio para el paquete principal cargado por adelantado y una instancia de servicio para cada módulo cargado con retraso.

## Experiencia mejorada del desarrollador

Ivy permite que _Angular Language Service_ admita comprobaciones adicionales durante el desarrollo. Esta es una gran mejora para la experiencia del desarrollador.

### Comprobaciones de ruta de archivo

_Angular Language Service_ verifica continuamente la hoja de estilo de componentes y las rutas de plantilla.

### Verificaciones de tipo de plantilla

Las plantillas se verifican por tipo, de acuerdo con el modo de verificación de tipo de plantilla como se describe más abajo en la sección "Modo estricto". Los nombres y tipos de miembros se verifican, incluso en vistas incrustadas. Lo que anteriormente resultaba en errores de tiempo de ejecución ahora se detecta durante el desarrollo y la construcción.

## Nueva API de depuración en modo de desarrollo

`ng.probe` ha sido reemplazado por una nueva API de depuración en modo de desarrollo. Las funciones más notables son `ng.applyChanges` y` ng.getComponent`.

## Modo estricto

### Esquema estricto del espacio de trabajo

El comando `ng new` ahora es compatible con el indicador` --strict` que por defecto está desactivado (`false`).

```bash
ng new my-app --strict
```

Cuando está habilitado, este parámetro agrega algunas comprobaciones estrictas del compilador TypeScript como se ve en el Listado 4.

```json
// tsconfig.json
{
  "compilerOptions": {
    "noImplicitAny": true,
    "noImplicitReturns": true,
    "noImplicitThis": true,
    "noFallthroughCasesInSwitch": true,
    "strictlyNullChecks": true
  }
}
```

*Listado 4. Opciones del compilador TypeScript habilitadas en un estricto espacio de trabajo angular.*

Curiosamente, esto no agrega las mismas opciones que si simplemente estableciéramos `"strict": true` en el objeto `compilerOptions`. Comparemos la opción estricta del workscpace Angular con la opción estricta del compilador TypeScript.

Ambos tienen estas opciones en común:

- `noImplicitAny`
- `noImplicitThis`
- `strictlyNullChecks`

La opción estricta de Angular establece además estas opciones:

- `noImplicitReturns`
- `noFallthroughCasesInSwitch`

mientras que la opción estricta del compilador TypeScript establece adicionalmente estas opciones:

- `alwaysStrict`
- `strictlyBindCallApply`
- `strictlyFunctionTypes`
- `estrictoPropertyInitialization`

Además, la opción de Angular no establece la comprobación del tipo de plantilla en el nuevo modo estricto, lo deja en el modo `'full'` anterior.

### Verificación estricta del tipo de plantilla

Hemos tenido la opción de habilitar la verificación del tipo de plantilla desde Angular versión 5 configurando `"fullTemplateTypeCheck": true` en el objeto` angularCompilerOptions`.

Ivy presenta una estricta comprobación del tipo de plantilla como se ve en el Listado 5. Cuando se establece esta nueva opción del compilador Angular, se ignora el valor de `fullTemplateTypeCheck`.

```typescript
// tsconfig.json
{
  "angularCompilerOptions": {
    "strictlyTemplates": true
  }
}
```

*Listado 5. Habilita la comprobación estricta del tipo de plantilla.*

La comprobación de tipo de plantilla estricta verifica los tipos de enlaces de propiedades y respeta la opción `stricNullChecks`. También verifica los tipos de referencias de plantilla a directivas y componentes, incluidos los tipos genéricos. También se verifican los tipos de variables de contexto de plantilla, lo cual es excelente para los bucles 'NgFor'. El tipo `$ event` está marcado para enlaces de eventos y animaciones. Incluso el tipo de elementos DOM nativos se verifica con una comprobación estricta del tipo de plantilla.

Estas comprobaciones adicionales pueden conducir a errores y falsos positivos bajo ciertas circunstancias. Por ejemplo, cuando se usan librerías que no están compiladas con `strictNullChecks`. Para abordar esto, la verificación estricta del tipo de plantilla tiene opciones para optar por no participar y ajustar los controles. Por ejemplo, `strictTemplates` es en realidad una atajo de 8 opciones de compilador Angular.

## Herencia de clase de componente y directiva mejorada

Las clases base sin selector ahora son compatibles con directivas y componentes. Algunos metadatos ahora se heredan de componentes base y clases directivas. Esto hace que sea más fácil extender, por ejemplo, las directivas Angular Components y Angular Router.

## Últimas versiones de TypeScript

Las versiones de TypeScript 3.6 y 3.7 son compatibles con la versión angular 9. Las versiones anteriores de TypeScript ya no son compatibles. Consulta la Tabla 1 para comparar la compatibilidad de TypeScript entre todas las versiones Angular.

[ver sin formato](https://gist.github.com/LayZeeDK/c822cc812f75bb07b7c55d07ba2719b3/raw/ad300b4d2bdbc51b4dd8092c059a68cb22d3774d/angular-cli-node-js-typescript-support -cs-type-script-node-type-script-code-node-code-node-type. csv](https://gist.github.com/LayZeeDK/c822cc812f75bb07b7c55d07ba2719b3#file-angular-cli-node-js-typescript-support-csv) alojado con ❤ por [GitHub](https://github.com/)

*Tabla 1. Tabla de compatibilidad de CLI angular, Angular, Node.js y TypeScript.* [*Abrir en una pestaña nueva*](https://gist.github.com/LayZeeDK/c822cc812f75bb07b7c55d07ba2719b3)*

TypeScript versión 3.6 presenta estas y otras características:

- Soporte Unicode para identificadores en destinos modernos
- Experiencia de desarrollador mejorada para promesas
- Comprobación de tipo más estricto de generadores

TypeScript versión 3.7 presenta estas y otras características que podemos usar con Angular versión 9:

- Operador de encadenamiento opcional (`?.`) similar al operador de navegación segura para plantillas Angular
- Operador de unión con nulos (`??`)
- Funciones de afirmación de pruebas (`assert parameterName is typeName` y `asserts parameterName`)
- `await` de primer nivel
- Alias ​​de tipo recursivo mejorado
- Mejora de la experiencia del desarrollador para funciones de comprobaciones

## Generación mejorada del lado del servidor con Angular Universal

Angular Universal versión 9 se lanza con un servidor de desarrollo Node.js Express para proporcionar un entorno realista durante el desarrollo.

También parte de este lanzamiento es un constructor del CLI para pre-renderizar rutas estáticas usando `guess-parser`, inspirado en `angular-prerender`. Podemos pasar un archivo de rutas para renderizar rutas dinámicas (rutas con parámetros).

### ¿Cómo empiezo?

Podemos agregar Angular Universal usando el comando `ng add @nguniversal/express-engine`. Luego podemos usar el comando del constructor `ng run myapp:serve-ssr` para iniciar el servidor de desarrollo de renderizado del lado del servidor con recarga en vivo. De manera similar, podemos usar `ng run myapp:prerender` para detectar rutas estáticas y dinámicas y pre renderizarlas.

## Experiencia de clases y estilo mejorada

Al aplicación de estilo en Angular Ivy ha sido reelaborado. La combinación de clases HTML estáticas con las directivas `NgStyle` y` NgClass` es ahora totalmente compatible y más fácil de entender.

### Soporte de propiedades personalizadas de CSS

Como parte de la reescritura del estilo Ivy, ahora se admiten las propiedades personalizadas de CSS enlazadas.

Un ejemplo de enlace css puede ser así:

```html
    <div [style.--my-var]="myProperty || 'any value'"></div>
```

Las propiedades personalizadas de CSS tienen control de alcance, por lo que esta propiedad de CSS se limitaría al DOM del componente.

## Lanzamiento estable de Bazel como opción de suscripción

Bazel versión 2.1 es una herramienta de automatización de compilación opcional para Angular versión 9.

### ¿Cómo empiezo?

Para habilitar Bazel, usa `ng add @angular/bazel` o la colección de esquemas `@angular/bazel` cuando generes un espacio de trabajo Angular.

Asegúrate de seguir [la guía de instalación de Bazel](https://docs.bazel.build/versions/2.0.0/install.html) para tu sistema operativo.

## Componentes Angular

La versión Angular 9 viene con componentes oficiales para YouTube y Google Maps. Se agregó una directiva y un servicio de portapapeles al Angular CDK.

## Pruebas

La mayor sorpresa de la versión Angular versión 9 son las muchas mejoras en las pruebas. Se resuelven problemas de rendimiento, se mejoran los tipos y se introducen nuevos conceptos.

[Obtén información sobre las principales características y mejoras para las pruebas en "Pruebas de siguiente nivel en Angular Ivy versión 9"](https://indepth.dev/next-level-testing-in-angular-ivy-version-9/).

## Conclusión

Uno de los objetivos más importantes ha sido mantener la compatibilidad hacia atrás entre Ivy y View Engine tanto como fue posible.

Por supuesto, la versión Angular 9 también incluye correcciones de errores, temas obsoletos y cambios importantes. Ivy también aborda algunos problemas de antiguos que no cubrimos en este artículo.

Angular Ivy es un facilitador para las características que están por venir. Como hemos discutido en este artículo, Ivy ya nos ha dado beneficios para diferentes casos de uso. Sin embargo, las mejores características llegarán en futuras versiones de Angular. ¿Cuál de las posibles características que se entregarán en las versiones 10 y 11 de Angular está por decidir?

Solo discutimos qué es parte de las API públicas y estables de Angular versión 9. Algunas API experimentales son parte de esta versión, como `renderComponent`,` markDirty` y `detectChanges`. Sin embargo, todavía están sujetos a cambios.

Con la retirada de las declaraciones de componentes de entrada y los componentes con carga diferida utilizando módulos de representación, estamos un paso más cerca de [componentes que se pueden eliminar del árbol y módulos Angular opcionales](https://indepth.dev/angular-revisited-tree-shakable-components- y-opcional-ngmodules /).

[Los componentes de funcionalidad](https://indepth.dev/component-features-with-angular-ivy/) también forman parte de esta versión, pero solo están expuestos para uso interno de Ivy.

La versión de Angular Ivy versión 9 nos brinda mejoras para el empaquetamiento, las pruebas, la experiencia del desarrollador, las herramientas, la depuración y la verificación de tipos. Toda una buena colección de características.

## Recursos relacionados

### Componentes con carga lenta

[Información sobre los módulos de presentación en la charla "Angular revisited: Tree-shakable components and optional NgModules"](https://youtu.be/DA3efofhpq4).

[Aprende a usar los componentes de carga diferida en "Lazy load components in Angular" por Kevin Kreuzer](https://medium.com/angular-in-depth/lazy-load-components-in-angular-596357ab05d8).

### Comprobación de tipos en plantillas

[Lee la guía oficial sobre la verificación de tipos en plantillas Angular para conocer los detalles de la resolución de problemas y la configuración](https://angular.io/guide/template-typecheck#troubleshooting-template-errors).

### Globalización

[Manfred Steyer analiza las configuraciones regionales de carga diferida en "Lazy Loading Locales with Angular"](https://www.softwarearchitekt.at/aktuelles/lazy-loading-locals-with-angular/).

[Cédric Exbrayat analiza la globalización de Ivy en "Internationalization with @angular/localize"](https://blog.ninja-squad.com/2019/12/10/angular-localize/).

### Ámbitos adicionales de proveedor de dependencias

Obtén información sobre los ámbitos de proveedor de `'any'` y `'platform'` en [“Improved Dependeny Injection with the new providedIn scopes ‘any’ and ‘platform’” by Christian Kohler](https://dev.to/christiankohler/improved-dependeny-injection-with-the-new-providedin-scopes-any-and-platform-30bb).

### Nueva API de depuración
[Lee sobre la API de depuración completa en la documentación oficial](https://angular.io/api/core/global).

### Angular Universal versión 9

Estos dos artículos entran en detalles de Angular Universal versión 9:

-   [“Angular Universal v9: What’s New ?” by Mark Pieszak](https://trilon.io/blog/angular-universal-v9-whats-new)
-   [“Angular v9 & Universal: SSR and prerendering out of the box!” by Sam Vloeberghs](https://dev.to/angular/angular-v9-universal-ssr-and-prerendering-out-of-the-box-33b1)

Aprende sobre `angular-prerender`, la biblioteca que inspiró estas nuevas características de Angular Universal en [“Prerender Angular Apps with a single Command” by Christoph Guttandin](https://media-codings.com/articles/prerender-angular-apps-with-a-single-command).

### Enlace de propiedades personalizadas de CSS

[Mira este tweet y la demostración de Alexey Zuev para ver los enlaces de Propiedades personalizadas de CSS en acción](https://twitter.com/yurzui/status/1221159415820275717).

## Revisores

Siempre es útil tener una segunda opinión sobre nuestro trabajo o incluso detectar errores tontos. Para este artículo tuve el placer de ser revisado por:

-   [Christoph Guttandin](https://twitter.com/chrisguttandin)
-   [Evgeny Fedorenko](https://indepth.dev/author/evgeny/)
-   [Santosh Yadav](https://dev.to/santoshyadav198613)

Lars Gyrup Brink Nielsen