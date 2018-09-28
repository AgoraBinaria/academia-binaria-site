---
title: Detección del cambio en Angular
permalink: deteccion-del-cambio-en-Angular
date: 2018-09-28 12:09:27
tags:  
- Angular
- Angular7
- Angular6
- Angular5
- Angular2
- ChangeDetection
- Tutorial
- Avanzado
categories:
- [Tutorial, Angular] 
thumbnail: /css/images/angular-9_change.png
---

![deteccion-del-cambio-en-Angular](/images/tutorial-angular-9_change.png)

La forma en que Angular realiza su renderizado y actualiza sus vistas es un factor clave para el rendimiento de las aplicaciones. ¿Cómo funciona la estrategia de detección de cambios de Angular? Pues tiene dos modos: `default` o *automágico* y `onPush` o *mindfullness*.

Es importante tomar consciencia porque es costoso realizar la detección más veces de lo necesario y porque no hacerlo suficientemente implica no ver resultados reales. Con un mayor conocimiento del `changDetectionStrategy` y un poco de trabajo extra tendrás aplicaciones más eficientes y mantenibles.


<!-- more -->

Partiendo de la aplicación tal cómo quedó en [Formularios reactivos con Angular](../formularios-reactivos-con-Angular/). Al finalizar tendrás una aplicación que actualiza la vista sólo cuando es necesario, es decir: cuando los datos han cambiado.

> Código asociado a este artículo en _GitHub_: [AcademiaBinaria/AutoBot/9-change](https://github.com/AcademiaBinaria/autobot/tree/9-change)


# 1 Comunicación de datos entre componentes

La detección de cambios se dispara ante eventos que le ocurren a los componentes. **La detección se realiza componente a componete**, así que compensa tener muchos componentes pequeños, para que cada uno por si sólo no genere demasiado ruido.

## 1.1 Componentes Contenedores y Presentadores

Al pasar de un único componente a varios mini-componentes, se propone usar **el patrón contenedor / presentadores**. Se mantiene un componente padre que contiene múltiples componentes presentadores hijos. El contendor es el único con acceso a los servicios de negocio y datos. Los presentadores reciben los datos y emiten eventos. Los presentadores no obtienen ni modifican por su cuenta.

Nomenclatura
- **Container**: aka *Parent, Smart* 
- **Presenter**: aka *Child, Dumb*

> Este reparto de responsabilidades es aconsejable independientemente de la estrategia de detección aplicada.

# 2 Change detectionstrategies

Con la aplicación bien estructurada en componentes y con la comunicación estandarizada, habremos reducido el impacto de la detección del cambio y estaremos preparados para optimizarlo. Conozcamos en detalle las estratégias de detección del cambio.

El decorador `@Component()` admite en su configuración la poco conocida propiedad `changeDetection`. Que de forma explícita se usa así:

``` typescript
import { ChangeDetectionStrategy, Component } from '@angular/core';
@Component({
  changeDetection: ChangeDetectionStrategy.Default,
  selector: 'app-root',
  template: `<h1>Changes are wellcome</h1> `
})
export class AppComponent {}
```

# 2.1 Detección automática, *default*

Por defecto, Angular tiene que ser conservador y verificar cada posible cambio, esto se denomina comprobación sucia o *dirty checking*. Se dispara **con demasiada frecuencia**, al menos en los siguientes casos: 

- Eventos desde el browser
- Timers, intervals etc..
- Llamadas http
- Promesas y código asíncrono.

Por si fuera poco, además de dispararse mucho es **muy costoso**. Determinar que algo ha cambiado implica comparar dos estados: el actual y el anterior.

> La comparación es valor a valor, en profundidad, para cada objeto de cada array, para cada propiedad de cada objeto.

Contod, esta estrategia es cómoda para el programador y suficiente para casos básicos. Pero demasiada mágia dificulta el control en aplicaciones complejas. Y en pantallas de mucha información e interacción degrada el rendimiento percibido.

# 2.2 Detección manual, *onPush*

Como se puede preveer, la detección del cambio manual es lanzada por el programador. No siempre va a ser laborioso, pero será más consciente pues para que ocurra han de darse alguna de estas circunstancias:

- **Explícitamente** el programador solicita la detección llamando a `ChangeDetectorRef.detectChanges();
- **Implícitamente** al usar el `pipe Async` en la vista se llama a ese mismo método.
- **Conscientemente** el desarrollador obliga a un componente a repintarse si le cambia la referencia a un `@Input()`.

> En este modo los componentes dejan de evaluar y comparar sus propiedades rutinariamente. Sólo atienden a eventos `@Output()` o **cambios de referencia** `@Input()`. Esto relaja mucho al motor de Angular, que ya no tiene que hacer comparaciones odiosas. Sabrá que algo ha cambiado porque... es otro objeto. 

# 3 Inmutabilidad

Como ya se ha dicho, para que Angular en la estrategia automática decida que algo ha cambiado necesita hacer una comparación por valor. Para evitar ese coste usamos la estrategia manual y el programador tienen que cambiar la referencia de algo cuando quiera que Angualar repinte la vista. 

## 3.1 Por referencia y por valor

Normalmente tendrá que crear un nuevo objeto y reasignarlo en lugar del anterior en un **ciclo de clonación, mutación y asignación**. Por costoso que parezca siempre compensa si evita muchas e innecesarias comparaciones por valor en estructuras profundas.

La estrategia `onPush` trata a todos los `Inputs` en inmutables, es decir, algo que no espera que cambie. Similar al paso de parámetros por valor, que si cambia es porque és otro puntero.

## 3.2 El clonado
El potencialmente pesado trabajo de clonado lo podemos evitar en muchos casos usando alguna de estas técnicas:

- **Tipos primitivos** que se pasan por valor en las propiedades `@Input()`
- **Arrays**: muchos métodos como `.filter() .slice() .sort() .concat()` etc, devuelven nuevas referencias sin modificar el array original.
- **Observables y el pipe Async**, pues en este caso se subscribe y lanza implícitamente la detección del cambio. Sin necesidad de clonar.

Para los demás casos tenemos operadores *TypeScript* sencillos y optimizados para obtener nuevas referencias a partir de otros ya existentes.

```typescript
const original = { name:'first', value:1 };
const cloned = { ...original };
const mutated = { ...original, value:2, newProperty: 'added' };
const list = [ original, cloned, mutated ];
const clonedList = [ ...list ];
const mutatedList = [ ...list, { name: 'new item'} ];
const newList = list.filter(i => i.name=='first');
```


Ya tienes los conocimientos para acelerar y reducir la incertidumbre sobre el actualización de vistas usando el patrón contenedor / presentador junto con la estrategia de detección de cambios OnPush.

> Para un ejemplo más completo de estos conceptos explora los componentes [Home de Autobot](https://github.com/AcademiaBinaria/autobot/tree/9-change/src/app/home). En [Car](https://github.com/AcademiaBinaria/autobot/tree/9-change/src/app/car) tienes un ejemplo de notificación manual usando `ChangeDetectorRef`.

Continúa tu formación avanzada para crear aplicaciones Angular siguiendo la serie del [tutorial avanzado de desarrollo con Angular](../tag/Avanzado/) y verás como aprendes a programar con Angular 7.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>
