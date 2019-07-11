---
title: 'npm: de instalador a publicador'
tags:
- NodeJS
- Avanzado
permalink: npm-de-instalador-a-publicador
id: 10
updated: '2016-04-19 07:01:25'
date: 2016-04-18 16:07:46
---

Seguro que estás harto de repetir la mítica instrucción `npm install paquete-x` Pero, ¿has pensado en **publicar tus propios módulos**?. Es muy recomendable crear soluciones distribuidas y muy fácil publicarlas en [*npm*](https://www.npmjs.com/).

Sigue este proceso y te convertirás en un creador de paquetes para *npm* :

## Programa

Todo empieza por tener una necesidad y resolverla **encapsulando su código en un módulo.** Por supuesto que ese módulo puede requerir paquetes externos. Y por supuesto que tu código puede, y debe, escribirse en varios módulos.

Antes de continuar vamos a fijar un par de conceptos:

**- módulo:** Fichero *.js* que exporta un funcionalidad y oculta su implementación.

**- paquete:** Uno o más módulos, con sus dependientes de otros paquetes que proponen una solución reutilizable.

Conocidos los ingredientes, la receta es sencilla:

1. Debes crear un proyecto para el paquete que vas a publicar.
2. Has de subir el código a un repositorio público.
3. Y debes rellenar convenientemente el `package.json` de tu proyecto.

Este es un ejemplo que he creado y [publicado en github](https://github.com/AgoraBinaria/ab-ftp) que te puede servir de base.

Presta especial atención al **nombrado** de tu proyecto porque ha de ser único en el repositorio de npm. Para reducir riesgos de colisiones de nombre te recomiendo que utilices un *prefijo* con tus iniciales o las de tu empresa.

No te olvides de incluir un juego de **pruebas y documentación** necesaria para sus uso.

## Publica

Puedes descargar librerías y herramienta desde el repositorio de *npm* de forma anónima. De hecho, eso es lo más corriente. Pero, como era de esperar, tienes que **registrarte para poder publicar** contenido. El [proceso de registro](https://www.npmjs.com/signup) vía web es sencillo y grátis.

Una vez verificado puedes hacer login en la web. Pero también en la terminal de tu ordenador: `npm login` te pedirá el nombre de usuario y contraseña. A partir de es momento estás identificado y puedes usar las herramientas de autor:

```
npm publish
```

Puedes encontrar [más información](https://docs.npmjs.com/cli/publish) acerca de este mega comando en la documentación de *npmjs*. Pero siguiendo la máxima de cuanto menos, mejor, la herramienta hará con sus valores por defecto un publicación y actualización limpias.

La clave está en que tu fichero de configuración `package.json` sea correcto y lo más completo posible. Escribe un completo `readme.md` que se convertirá en portada de tu paquete en el repositorio de npm. Mira en mi ejemplo cómo [el contenido subido a github](https://github.com/AgoraBinaria/ab-ftp) se transforma y se ve en la [página de información de npm.](https://www.npmjs.com/package/ab-ftp)


Comprueba en un directorio vacío que `npm install nombre-de-tu-paquete` descarga todo lo necesario... y tómate un café mientras ves crecer la hierba y las estadísticas de descargas.


## Actualiza

Con el tiempo mejorarás y corregirás tu solución. Te recomiendo que sigas el patrón de [nombrado de versiones semántico](http://semver.org/):

**x.y.z = 1.2.3 = ruptura.mejora.parche**

Es bueno que etiquetes tu repositorio con estas mismas versiones para que, una vez desplegadas, se encuentre con facilidad en *github*.

```
git tag 0.1.2
git push --tags
```


## Disfruta

Finalmente podrás utilizar ese código en todas tus aplicaciones. Sería fantástico que además otros lo encontrasen útil. Y mejor aún si te ayudan o al menos te proponen mejoras. Pero en cualquier caso **siempre habrás ganado**. El esfuerzo analítico que requiere se cobra con creces: dividiendo el problema, creando micro soluciones eficientes y reutilizando código siempre es rentable.