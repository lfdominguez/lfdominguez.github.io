---
layout: post
title: Asegurando nuestras comunicaciones con WireGuard
tags: [vpn, wireguard, linux, seguridad]
summary: Como instalar y configurar un servidor VPN con WireGuard
---

## Introducción

En muchos casos como SysAdmins debemos o queremos mantener comunicación con algún servidor desde un punto donde el canal de comunicación puede ser revisado (con sniffers) por otras personas o sistemas. Algunos ejemplos serían:

 * Administrar nuestros servidores desde Internet.
 * Darle acceso a usuarios a servicios sobre un canal inseguro.
 * Brindar seguridad ante sniffer en un canal independientemente de la seguridad de los servicios que brinde.
 * Queremos darle acceso a algún servicio de monitoreo al encargado de Seguridad que se encuentra fuera de la red de administración.
 * Entre otros...

Una de las soluciones que se presenta para este tipo de problema son las VPN (Virtual Private Network), que permiten crear una red dentro de otra y generalmente encriptada, por lo que los datos que se transmiten se ocultan a los ojos de los interesados, no importa que sean a servicios no seguros, pues se comunican a través del túnel creado por este servicio.

En la actualidad hay 2 herramientas que son ampliamente para este menester:

 * IPSec
 * OpenVPN

Ambos son muy buenos en esa tarea y gozan de un gran soporte por todos los equipamientos de red (IPSec más al ser el estándar de facto en muchos routers, como los de CISCO o HUAWEI). El problema a veces con éstos es que su instalación, configuración y puesta en marcha puede resultar un poco tediosa.

Hoy vengo con una propuesta que está ganando mucho terreno a pesar de ser un producto en desarrollo, pero que muchos le han puesto el ojo, es [WireGuard](https://www.wireguard.com) y supone una ventaja con respecto a los anteriores por:

  * Extremadamente fácil en su instalación y despliegue, ellos indican que *"(...) aims to be easy to configure and deploy as SSH (...)"*.
  * Usan criptografía *state-of-the-art* que no es más que la utilización de los algoritmos más modernos de éstos menesteres.
  * Al igual que IPSec es compilado como módulo del kernel a través de DKMS. Lo que logra un rendimiento mucho mayor que OpenVPN (pues este es *user-space*).
  * Usa los algoritmos de encriptación *Noise Protocol Framework, Curve25519, ChaCha20, Poly1305, BLAKE2, SipHash24, HKDF*.
  * Se ha implementado teniendo en cuenta menor cantidad de código, lo que permite una mejor auditoría de código y un menor área de ataque. Está pensado para que una persona pueda auditar todo el código sin complicaciones.
  * Entre otras ....

WireGuard por ahora no soporta la asignación dinámica de IP, por lo que cada cliente que tengamos se le debe establecer el ip en su configuración. Además que WireGuard no permite la comunicación de un cliente que no tenga el IP especificado en su fichero de configuración.

Algunas gráficas sobre el rendimiento de WireGuard con respecto a IPSec y OpenVPN pueden verse en [Performance of WireGuard](https://www.wireguard.com/performance/)

## Instalación

Me voy a centrar en la instalación sobre Debian, pues es lo que uso en mis servidores gg. Debido a que WireGuard es un proyecto en desarrollo, encontrarás los paquetes de su instalación en el repositorio de *sid* que es el repositorio "intestable":

  * [wireguard-dkms](https://packages.debian.org/sid/wireguard-dkms)
  * [wireguard-tools](https://packages.debian.org/sid/wireguard-tools)

Luego de haber bajado los paquetes *.deb* solo debemos instalarlos, por ej:

```terminal
# dpkg -i wireguard-dkms.deb wireguard-tools.deb
```
