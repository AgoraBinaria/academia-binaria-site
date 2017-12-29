---
title: Vigilancia y seguridad en Angular
permalink: vigilancia-y-seguridad-en-Angular
date: 2017-12-29 11:49:27
tags:  
- Angular
- Angular5
- Angular2
- observables
- Tutorial
- Introducción
categories:
- [Tutorial, Angular] 
thumbnail: /css/images/angular-5_7_watch.png
---

![Tutorial Angular5 6-http](/images/tutorial-angular-5_7_watch.png)

Las comunicaciones _http_ son una pieza fundamental del desarrollo web, y en **Angular** siempre han sido fáciles y potentes. ¿Siempre?, bueno cuando apareció Angular 2 echábamos en falta algunas cosillas. Pero con la versión actual **consumir un servicio REST** vuelve a ser cosa de niños.

Claro que para ello tendremos que jugar con los _observables_ y los servicios de la librería `@angular/common/http` con los que realizar **comunicaciones asíncronas en Angular**.

<!-- more -->

Partiendo de la aplicación tal cómo quedó en [Comunicaciones http en Angular](../comunicaciones-http-en-Angular/). Al finalizar tendrás una aplicación que almacena y recupera los datos consumiendo un servicio REST.

> Código asociado a este artículo en _GitHub_: [AcademiaBinaria/angular5/7-watch](https://github.com/AcademiaBinaria/angular5/tree/master/7-watch/cash-flow)
> El servicio REST se encuentra en _GitHub_: [AcademiaBinaria/ApiBase](https://github.com/AcademiaBinaria/ApiBase)

# 1 seguridad

```typescript
export class OperationsService {
  private url = environment.apiUrl + "pub/items/";

  constructor(private http: HttpClient) {}
}
```

# 2 vigilancia

> Cada método de negocio, configura la llamada de infraestructura; parece poca cosa. Podría ser un buen sitio para validar la información antes de ser enviada, o quizás agrupar varias llamadas de red para una misma operación de negocio. El _dolar_ al final del nombre es un convenio para las funciones que devuelven observables.

Ya tenemos los datos almacenados en un servidor con el que nos comunicamos por _http_; aunque por ahora de forma anónima. Con el conocimiento actual de los observables, el _httpClient_ y los interceptores estamos a un paso de darle seguridad a las comunicaciones. Repasa esta serie [tutorial de introducción a Angular](../categories/Tutorial/Angular/) verás como aprendes a programar con Angular5.

> Aprender, programar, disfrutar, repetir.
> -- <cite>Saludos, Alberto Basalo</cite>
