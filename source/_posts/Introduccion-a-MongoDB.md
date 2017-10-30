---
title: Introducción a MongoDB
tags: 
- MongoDB
categories:
- Introducción 
permalink: introduccion-a-mongodb
id: 3
updated: 2015/08/25 06:26:25
date: 2015/08/15 11:00:54
---

## ¿Qué es MongoDB?

[MongoDB ](http://mongodb.org) es una base de datos orientada a **documentos** con esquema dinámico. Esto le permite ofrecer un alto **rendimiento** y facilita el desarrollo de aplicaciones. A cambio nos impide tener **Joins y Transacciones** algo muy habitual en bases de datos relacionales, pero sin lo que se puede vivir.

### ¿De qué se compone?

Después de su instalación disponemos en un directorio de varios ejecutables que componen **la aplicación servidora y sus herramientas**. El ejecutable fundamental será `mongod` que es el servidor, es el motor de manejo de datos. Para acceder al servidor se nos ofrece una aplicación de consola llamada `mongo`. Este es un intérprete de JavaScript y con él, podemos comunicarnos con la base de datos.

### ¿Cómo funciona?

MongoDB almacena **documentos en formato JSON**. Bueno realmente lo hace en *BSON* que es un superconjunto de JSON. BSON es un formato binario que optimiza espacio, rendimiento y aporta funciones extra sobre JSON. Pero esencialmente para nosotros como usuarios toda la entrada salida es en JSON.

Como los documentos se almacenan en JSON y la consola es un interprete JavaScript, la verdad es que **la consola ofrece una potencia enorme** para realizar operaciones de inserción, selección y manipulación de datos. Cabe destacar que la consola es un programa e intérprete *síncrono*. Con algunos drivers como el de NodeJS el trabajo es puramente *asíncrono*.

### ¿Se parece a SQL?

Si vienes del mundo relacional, cuando llegues a Mongo tendrás que ajustar tu punto de mira al definir tus estructuras de datos. Como una primera guía te diré que hay una ligera **equivalencia entre Mongo y los SQL**

*SQL* -> **MongoDB**

*DataBase* -> **DataBase**

*Table* -> **Collection**

*Row* -> **Document**

*Field* -> **Property**

*Join* -> **Embedded**

*Index* -> **Index**


La enorme diferencia está en que **en una colección se pueden guardar documentos con esquemas distintos**, y que esos esquemas pueden incluir documentos complejos, como arrays y subdocumentos. A esta falta de rigor al definir el esquema se la conoce
en inglés como *schemaless*. Estamos ante una base de datos que almacena documentos de **esquema dinámico**.

Lo anterior es sin duda un valor diferencial a favor de Mongo pues facilita mucho el desarrollo y evolución de las aplicaciones. Otra cosa es la ausencia de Joins y Transacciones entre colecciones que obliga a diseñar y codificar de manera metódica para garantizar **eficiencia e integridad** en las consultas. La sugerencia es aprovechar sus fortalezas para minimizar las debilidades.

### ¿Y entonces?

**MongoDB** tienes campos de aplicación en los que supera a los sistemas relacionales en potencia y facilidad de desarrollo. Pero, hay que saber **escoger la herramienta mas adecuada** para cada caso. Ahora que ya sabes un poco de que va esto de MongoDB puedes empezar a buscar aplicaciones dónde seguro que le sentará como un guante.