---
layout: post
title: Montando devdocs.io en nuestra red (Docker incluido)
tags: [docker, desarrollo, documentación]
summary: Como instalar y configurar un servicio de devdocs.io, incluyendo Docker
---

## Introducción

Cuando vamos a programar un script o algo más grande, necesitamos documentarnos
acerca de las bondades que nos ofrece el lenguaje que escogimos para dicha tarea,
[devdocs.io](https://devdocs.io) es un sitio que reune mucha documentación sobre disímiles tecnologías,
agrupadas a veces por versiones. Es una muy buena recopilación, que permite incluso
descargarla offline para el almacenamiento del navegador, para que, una vez estemos
offline poder consultarla.

Lo bueno de este proyecto es que es libre, es decir, que podemos encontrar su código
fuente en [GitHub](https://github.com/freeCodeCamp/devdocs). De eso se trataremos, de
cómo instalar en nuestra red local este servicio, incluyendo al final como hacerlo si
prefieres Docker.

## Instalando en Debian

Una vez bajado el proyecto

```terminal
# apt install git
# git clone https://github.com/freeCodeCamp/devdocs.git
# cd devdocs
```

Lo primero que debemos hacer es instalar `ruby` y `bundle` para gestionar las
dependencias del proyecto:

```terminal
# apt install ruby bundle
```

Ahora procedemos a instalar algunos paquetes adicionales, para que cuando
usemos `bundle`, que compile nativamente algunas dependencias, no de errores.

```terminal
# apt install build-essential ruby-dev zlib1g-dev nodejs ruby-thor
```

Ahora obtenemos todas las dependencias necesarias para que funcione el proyecto

```terminal
# bundle install
```

Ya tenemos la base del proyecto, pero no tenemos ninguna documentación
descargada, para eso usaremos `thor`

```terminal
# thor docs:download --all
```

Ya el proyecto está listo para ser ejecutado y empezar a brindar el servicio,
el comando para arrancar el servidor es

```terminal
# rackup -o 0.0.0.0 -p 80 -D
```

donde `-o` indica por que IP escuchará el servidor, definiendo el puerto con
`-p`. El `-D` indica que se ejecute como un demonio.

Como generalmente pondermos esto en un servidor aparte (preferiblemente un
contenedor), pudieramos modificar directamente `/etc/rc.local` para que
ejecutara el comando, ejemplo de este fichero sería

```bash
#!/bin/bash

cd /opt/devdocs
rackup -o 0.0.0.0 -p 80 -D
cd -

exit 0
```

Con esto cada vez que se ejecute nuestro servidor, tendremos el servicio
de `devdocs.io` corriendo.

## Instalando con Docker

Si eres como yo, que ahora todo lo quiero dockerizar, puesto que he visto
muchas ventajas en una infraestructura de contenedores con Docker, devdocs te
brinda su Dockerfile para que lo construyas, los pasos son meramente sencillos

Instalamos Docker, en los repos de Debian y Ubuntu viene el paquete, en caso
de que no uses ninguno de estos, remitete a su documentación para la
instalación.

```terminal
# apt install docker.io
```

Teniendo Docker instalado, procedemos a descargar devdocs de su repositorio
y construirlo

```terminal
# apt install git
# git clone https://github.com/freeCodeCamp/devdocs.git
# cd devdocs

# docker build -t thibaut/devdocs .
```

Esperamos que se construya la imágen, de alguna manera hace todo lo que hicimos
en el apartado anterior, pero automatizado. Una vez que se haya construido la
imagen, solo tenemos que ejecutarlo

```terminal
# docker run --name devdocs --restart always -d -p 9292:9292 thibaut/devdocs
```

Explicando un poco, sería que corriera un contenedor nombrado con `--name`, que
se iniciara automáticamente (por `--restart always`), que corra como un
demonio (`-d`) y que el puerto con que se publicará será el 9292 (por la
primera parte del par `9292:9292`, la segunda parte es del servicio interno).
