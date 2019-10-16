---
title: Internacionalización y puesta en producción
permalink: internacionalizacion-y-puesta-en-produccion-con-Angular
date: 2018-10-17 09:50:27
tags:
- Angular
- Angular8
- Angular2
- i18n
- Tutorial
- Avanzado
categories:
- [Tutorial, Angular]
thumbnail: /css/images/angular-18_i18n.png
---

![internacionalizacion-y-puesta-en-produccion-con-Angular](/images/tutorial-angular-18_i18n.png)

Las aplicaciones web, son la expresión perfecta del mundo globalizado en el que vivimos. Preparar tu aplicación para ese mundo se conoce coo _internationalization_, o recortadamente **i18n**. Atender a las necesidades específicas de tus usuarios es el proceso de __localization__.

Para empezar esto afecta las traducciones de los contenidos: sean datos o textos fijos. Pero también a las adaptaciones culturales para la presentación de fechas, números, iconos... En Angular tenemos herramientas y soluciones para poner en marcha proyectos globalizados.

<!-- more -->

Partiendo del código tal como quedó en [Velocidad y SEO con el SSR de Angular Universal](../velocidad-y-seo-con-el-ssr-de-angular-universal/). Al finalizar tendrás una aplicación que adapta a la cultura del usuario.

> Código asociado a este tutorial en _GitHub_: [AcademiaBinaria/angular-boss](https://github.com/AcademiaBinaria/angular-boss)

# 1 Traducciones y contenido

## 1.1 xi18n


`apps\warehouse\src\app\app.component.html`

```html
<header>
  <h1 i18n>Welcome to the Angular Builders Warehouse</h1>
</header>
<img src="../assets/Warehouse-Building.jpg"
     alt="Warehouse building"
     i18n-alt>
<router-outlet></router-outlet>
<footer>
  <a href="https://angular.builders"
     target="blank">Angular.Builders: </a>
  <span i18n>a store of resources for developers and software architects.</span>
</footer>
```

`apps\warehouse\tsconfig.app.json`

```json
{
"angularCompilerOptions": {
    "enableIvy": false
  }
}
```

`package.json`

```json
{
  "i18n:warehouse": "ng xi18n warehouse --output-path src/locale",
}
```


## 1.2 Build configurations

`angular.json`

```json
"production-es": {
  "fileReplacements": [
    {
      "replace": "apps/warehouse/src/environments/environment.ts",
      "with": "apps/warehouse/src/environments/environment.prod.es.ts"
    },
  ],
  "outputPath": "dist/apps/warehouse/es/",
  "i18nFile": "apps/warehouse/src/locale/messages.es.xlf",
  "i18nFormat": "xlf",
  "i18nLocale": "es",
  "baseHref": "es",
}
```

```json
{
"build:warehouse-es": "ng build warehouse --configuration=production-es",
"start:warehouse-es": "npm run build:warehouse-es && angular-http-server --open -p 8082 --path ./dist/apps/warehouse/es",
}
```

# 2 Adaptaciones culturales de tiempo y moneda

## 2.1 Registro manual en app.module o Auto registro en angular.json

### Manual

```typescript
import { registerLocaleData } from '@angular/common';
import localeEs from '@angular/common/locales/es';

registerLocaleData(localeEs);
```

### Automático

```json
"i18nLocale": "es"
```

## 2.2 Tiempo, moneda y contenido

`apps\warehouse\src\app\app.component.html`

```html
<article class="card">
  <p>{{ building.date | date:'long' }}</p>
  <p>${{ building.value | number }}<i> {{ building.status }}</i></p>
</article>
```

`apps\warehouse\src\app\app.component.ts`

```TypeScript
public building = {
  date: Date.now(),
  value: 2345.897,
  status: 'buy'
};
constructor() {
  if (this.building.status === 'buy') {
    this.building.status = environment.buy;
  } else {
    this.building.status = environment.sell;
  }
}
```

```TypeScript
{
  buy: 'for buy',
  sell: 'for sell'
}
{
  buy: 'para comprar',
  sell: 'para vender'
}
```

Ahora ya tienes una aplicación que satisface a usuarios y robots por igual. Continúa tu formación avanzada para crear aplicaciones Angular siguiendo la serie del [tutorial avanzado de desarrollo con Angular](../tag/Avanzado/) y verás como aprendes a programar con Angular 8.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>