---
title: Velocidad y SEO con el SSR de Angular Universal
permalink: velocidad-y-seo-con-el-ssr-de-angular-universal
date: 2018-10-30 12:50:27
tags:  
- Angular
- Angular7
- Angular6
- Angular5
- Angular2
- Universal
- SSR
- Tutorial
- Avanzado
categories:
- [Tutorial, Angular] 
thumbnail: /css/images/angular-c_universal.png
---

![velocidad-y-seo-con-el-ssr-de-angular-universal](/images/tutorial-angular-c_universal.png)

Las *SPA JavaScript*, muy balanceadas hacia el navegador, nacieron para **crear con tecnología web aplicaciones de negocio**. Normalmente se desplegaban en *intranets*, o en internet para usuarios autorizados. Eran aplicaciones de uso intensivo, visita recurrente y alto rendimiento diario. El éxito tecnológico de *frameworks* como Angular las llevó a ser usadas para desarrollar webs clásicas de internet y ser utilizadas por visitantes ocasionales. 
 
Pero en esta situación presentaron dos problemas para los que inicialmente no estaban preparadas. Por un lado la primera visita de un humano obligaba a la descarga completa de la aplicación antes de poder ver nada. Y nada era lo que veían los visitantes robóticos que pretendían indexar un sitio. Las soluciones a estos problemas incluyen, entre otras medidas, **una vuelta al servidor**. Lo que en Angular se conoce como **aplicación universal**. 


<!-- more -->

Partiendo del código tal cómo quedó en [El patrón Redux con NgRx en Angular](../el-patron-redux-con-ngrx-en-angular/). Al finalizar tendrás una aplicación que se instala, actualiza y comporta como una aplicación nativa.

> Código asociado a este artículo en _GitHub_: [AcademiaBinaria/AutoBot/c-universal](https://github.com/AcademiaBinaria/autobot/tree/c-universal) y en el proyecto demo [AcademiaBinaria/Astrobot](https://github.com/AcademiaBinaria/astrobot)

# 1 Velocidad, primera visita y sucesivas

El reto está en mantener lo bueno de las aplicaciones *javascript* como es la transición fluida entre páginas, la interactividad o la descarga de datos bajo demanda. Pero combinado con una mejor primera experiencia. Para ello la descarga del `index.html` tiene que venir con un documento html ya preparado con algo para mostrar y tardar lo menos posible en permitir la interacción. 

El **tiempo para el primer pintado** se ve penalizado por el tamaño del *bundle* principal de Angular, pues en él reside habitualmente el componente *app* que actúa de raiz. Por supuesto que utilizar la carga diferida de módulos es una manera obligada de reducir el peso del *main*. Todas las rutas, incluída la ruta base, deben ser *lazy* para retrasar la navegación y que la descarga se produzca más tarde. Sólo el componente raiz con la *shell* de navegación básica debería venir en el *bundle* principal.

Pero ni con eso es suficiente. El usuario no verá nada hasta que Angular se descargue, reclame el *bundle main*, lo procese y renderece ese *shell*. Hay usar alguna estrategia extra para reducir el tiempo de espera y entretener al usaurio.

## 1.1 Prerenderizado estático

La más sencilla es hacer que el `index.html`, habitualmente vacío, se rellene con un **contenido visualizable mientras el proceso principal de Angular no arranca**. En ocasiones basta con poner a mano algún mensaje o animación que indique que estamos cargando. Pero cuanto más se parezca esa primera visión al resultado final mejor para el usuario. Así que lo propio sería que el `index.html` ya bajase con el *shell* real de la aplicación.

Montar eso a mano no es la mejor opción. La solución parte de **renderizar el *html* durante el proceso de *deploy***. Se trata de configurar el CLI para que ejecute la aplicación en la máquina del desarrollador sobre una ruta predefinida; y que copie el resultado sobre el `index.html` que enviará a distribución.

Este trabajo se ha automatizado y se resuelve con un par de comandos del Angular CLI. 

```bash
ng g app-shell --client-project astrobot --universal-project server-astrobot
```

El efecto de este comando se materializa especialmente con la aparición de nuevos *targets para los builders* del CLI en el fichero [angular.json](https://github.com/AcademiaBinaria/astrobot/blob/fff2a9f2c38f4b333489063b6f812a3b614fe173/angular.json#L141). 

Con el comando `ng run astrobot:app-shell` podrás generar una versión especial de distribución en al que el `index.html` ya va prerenderizado con el contenido del componente asociado a la ruta *shell*. Realmente lo que hace es ejecutar tu aplicación sobre una ruta predefinida, tomar el *html* resultado e inyectarlo en el *body* del `index.html` que irá a distribución.

Compruébalo en la ejecución de [Astrobot](https://academiabinaria.github.io/astrobot/), viendo el código fuente de la página descargada.

> Por cierto, esta técnica no obliga a disponer de ningún servidor web especial. Sigue funcinando con un servidor estático de ficheros pues la prerenderización se produjo en la máquina del programador.

## 1.2 Renderizar en el servidor con SSR

Claro que esto es sólo un truco para que ese primer momento de espera se reduzca y no perdamos potenciales visitantes. Si queremos algo más, como por ejemplo que el contenido a descargar sea más fresco, entonces necesitaremos **renderizar en el servidor**. Y para ello necesitaremos un servidor de verdad. El propuesto y mejor documentado es *Express de NodeJS*.

Para empezar tendrás que instalar y registrar las librerias necesarias. Además habrá que crear el pequeño servidor *Express*, y configurar al CLI para que haga el despliegue de ambos: cliente y servidor. Este laborioso trabajo se ha automatizado y ahora mismo se resuelve casi todo con un par de instrucciones.

```bash
ng add @nguniversal/express-engine
npm run build:ssr && npm run serve:ssr
```

El resultado es un servidor *Node* que a cada petición web responde enviando el `index.html`. Pero, y esta es la clave, resolverá la ruta ejecutando la aplicación Angular antes de responder al navegador. De esa forma el `index.html` irá recién generado con el contenido tal cual lo vería el usuario tras la ejecución de Angular en local. Así que **la espera al primer pintado significativo se reduce** y eso es bueno.

Por si fuera poco, la principal ventaja al usar este método es que al traer información dinámica puede usarse para indexar el contenido real del sitio. Esto es dóblemente bueno, porque ahora todos **los robots indexadores podrán catalogar tu *site*** como si de una web clásica se tratase. Y los usuarios humanos podrán continuar la ejecución en local disfrutando de las ventajas de una SPA.

De todas formas tengo que advertirte de que tomes todo esto con cautela por varios motivos:
- Tecnología compleja estable pero con carencias
- Herramientas de generación buenas pero incompletas
- Transferencia de estado manual para evitar llamadas repetidas al API

Tampoco es sencilla la convivencia con librerias propias del *browser*, y menos si se trata de una PWA. Yo procuro usar un servicio que aisle al servidor de ciertas llamadas sólo disponibles en el navegador, como por ejemplo todo lo relativo al *localStorage*.

```typescript
import { isPlatformBrowser, isPlatformServer } from '@angular/common';
import { Inject, Injectable, PLATFORM_ID } from '@angular/core';
@Injectable({
  providedIn: 'root'
})
export class UniversalService {
  constructor(@Inject(PLATFORM_ID) private platformId: string) {}
  public isBrowser() => isPlatformBrowser(this.platformId);
  public isServer() => isPlatformServer(this.platformId);

  public saveOnStorage(key, value) {
    if (this.isBrowser()) {
      sessionStorage.setItem(key, value);
    } else {
    }
  }
  public loadFromStorage(key) {
    if (this.isBrowser()) {
      sessionStorage.getItem(key);
    } else {
      return null;
    }
  }
}
```

# 2 SEO en la página, en el navegador y en el servidor

Con lo visto hasta ahora tu apliación estará más que cubierta en cuanto a ofrecer la mejor experiencia para usuarios al tiempo que envía contenido indexable para robots... Pero falta algo.

Habitualmente las aplicaciones Angular manejan el contenido visible de una página web; es decir, el *body*. Para **acceder y cambiar el contenido del *header***, tan utilizado por los robots de redes sociales, hay que usar algo más.

## 2.1 Titulo y meta etiquetas de página

Como parte del *framework* viene la librería *platform-browser* dónde tenemos un par de servicios para manipular el título y cualquier etiqueta de meta información de la página.

Para ello suele usarse un código similar a este en el componente raiz de la aplicacion. Es muy sencillo pero te dará una idea del potencial de estos servicios:

```typescript
import { Meta, Title } from '@angular/platform-browser';
@Component({
  selector: 'app-root',
  template: `<p>Aprende a usar el framework Angular</p>`,
})
export class AppComponent implements OnInit {
  constructor(private title: Title, private meta: Meta) {}
  ngOnInit() {
    this.meta.addTag({ property: 'og:title', content: 'My title' }, true);
    this.meta.updateTag({ property: 'og:title', content: 'My title' });
  }
}
```

Ahora ya tienes una aplicación que satisface a usuarios y robots por igual. Continúa tu formación avanzada para crear aplicaciones Angular siguiendo la serie del [tutorial avanzado de desarrollo con Angular](../tag/Avanzado/) y verás como aprendes a programar con Angular 7.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>