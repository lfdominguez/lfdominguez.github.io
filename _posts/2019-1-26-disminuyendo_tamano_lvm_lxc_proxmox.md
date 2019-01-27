---
layout: post
title: Disminuyendo tamaño de HDD virtual de un LXC en Proxmox
tags: [proxmox, lxc, lvm]
summary: Pasos para disminuir el tamaño de un HDD virtual de un contenedor LXC en Proxmox.
---

## Introducción

Proxmox es la plataforma de virtualización (basada en Debian) que utilizo (hasta dentro de poco, pues me mudo a Docker Swarm ;)). En un momento determinado tuve la necesidad de mover uno de los HDD virtuales de uno de mis contenedores LXC para otro espacio de almacenamiento más reducido. El HDD virtual estaba definido con un tamaño mayor al que a donde se migraría dicho HDD. Al revisar internamente el espacio ocupado de dicho HDD, era mucho menor al definido, por lo que la solución sería disminuir el HDD virtual para que pudiera ser migrado.

## Reducción del HDD

Importante aclarar que los pasos siguientes son para cuando el formato del sistema de ficheros es ext2, ext3 o ext4.

Al utilizar la propia herramienta de Proxmox para ello, en la documentación define que se utiliza solamente para el aumento del tamaño, por lo que la solución fue utilizar los comandos base para dicha tarea:

  1. Apagamos el contenedor que queremos migrar.
  2. Accedemos por la terminal al nodo Proxmox donde se encuentra el contenedor.
     * Obtenemos el HDD que queremos disminuir con:
     ```terminal
     $ df -h
     ```
  3. Comprobamos el HDD virtual para verificar que no contenga errores:
  ```terminal
  e2fsck -fy <HDD-VIRTUAL>
  ```
  4. Reducimos el sistema de ficheros en sí:
  ```terminal
  resize2fs <HDD-VIRTUAL> <Tamaño final - 1><Unidad>
  ```
Explicando un poco, cuando se reduce (o amplía) un sistema de ficheros, en realidad son varios los pasos a realizar. Primero se debe reducir el sistema de ficheros, para que la información que se encuentra esparcida por el HDD se quede en los ámbitos del nuevo espacio definido para que luego cuando se disminuya el espacio de la partición, no exista pérdida de datos.
En cuanto a la expresión `<Tamaño final - 1>` es debido a que como se utilizan programas distintos para el cambio de tamaño del sistema de ficheros y la partición, éstos pueden tener distintas definiciones internas de la unidad de medida y por tanto se deja un espacio de seguridad.
En cuanto a la expresión `<Unidad>` se refiere a la unidad de medida del nuevo espacio:
  * `M` Megabyte
  * `G` Gigabyte
  * `T` Terabyte
  * etc.

  5. Reducimos la partición LVM:
```terminal
lvreduce -L <Tamaño final><Unidad> <HDD-VIRTUAL>
```
  6. Ahora aumentamos el sistema de archivos automáticamente hasta el tamaño final de la partición.
```terminal
resize2fs <HDD-VIRTUAL>
```
  7. Cambiamos manualmente la definición de la máquina virtual para que refleje el nuevo tamaño del HDD virtual
```terminal
nano /etc/pve/lxc/<id>.conf
```
Una línea que contiene
```
rootfs: <id del hdd>,size=<tamaño>
```
En tamaño ponemos la misma definición que establecimos anteriormente en el comando `lvreduce`.