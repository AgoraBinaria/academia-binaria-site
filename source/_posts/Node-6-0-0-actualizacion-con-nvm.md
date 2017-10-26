---
title: Node 6.0.0 actualización con nvm
tags:  
- NodeJS
categories:
- Introducción 
permalink: node-6-0-0-actualizacion-con-nvm
id: 11
updated: '2016-05-05 10:02:26'
date: 2016-05-05 09:39:40
---

Acabamos de recibir la buena noticia de la versión 6 de **NodeJS**. En este caso con mejoras de rendimiento e incorporación de sintaxis de **ES6**. ¿Cómo obtener esta nueva versión?. Y sobretodo,  ¿cómo manejar la convivencia de distintas versiones? La ayuda se llama [nvm](https://github.com/creationix/nvm) 

Estos son los pasos que has de seguir para instalar la herramienta.

Se recomienda desinstalar las versiones de **node** y **npm** instaladas previamente. No es obligatorio.

#### LINUX & OS X

1- Comprobamos que tenemos instaladas las dependencias
1.1 - Dependencias **Linux**
```
sudo apt-get update
sudo apt-get install build-essential
```

1.2 -Dependencias **OSX** (herramientas de linea de comandos para XCode)
```
xcode-select --install
```


2- Descargamos y ejecutamos el script de instalación

```	
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.31.0/install.sh | bash
```


3- Comprobamos que está instalado **NVM**
```
command -v nvm 
```
*debe devolver nvm*


4- Instalamos la versión de Node que deseamos
```	
nvm install 6.0.0
```

5- Elegimos la versión de Node instalada que deseamos usar

```	
nvm use 6.0.0
```
	
#### WINDOWS

Para **Windows** no existe una versión *nativa* de **nvm**, pero si hay dos opciones para gestionar las versiones de node instaladas.

1- **nvm-windows**
	https://github.com/coreybutler/nvm-windows
	Es un wrapper de npm para windows, el instalador se encuentra en la URL (https://github.com/coreybutler/nvm/releases).
	
Los `comandos` para instalar y la versión de Node son los mismos que en Linux y OSX.
	
2- **nodist**  
Es un gestor de versiones parecido a NVM solo para Windows
	La URL del instalador (https://github.com/marcelklehr/nodist/releases/download/v0.7.2/NodistSetup-v0.7.2.exe).
	Los comandos se encuentran en la documentación. (https://github.com/marcelklehr/nodist)