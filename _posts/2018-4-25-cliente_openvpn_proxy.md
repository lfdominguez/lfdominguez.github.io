---
layout: post
title: Usando el cliente de OpenVPN a través de un proxy
tags: [vpn, openvpn, linux, seguridad, proxy]
summary: Cómo usar el cliente de OpenVPN cuando estamos detrás de un proxy HTTP
---

> **Disclaimer**: Solo para uso educacional, no me hago responsable por el uso que se le de al presente manual.

## Introducción

En esta ocasión no explicaré como se instala, configura y despliega un servidor de OpenVPN, pues eso es para otro post, o de alguien más. En este caso mostraré como usar el cliente de OpenVPN cuando nos encontramos detrás de un proxy (la mayoría de los sysadmin que conozco).

A veces como sysadmin nos encontramos con la necesidad de acceder a un servicio, por ejemplo el de actualizar las llaves públicas de nuestro sistema con GPG, que necesita hacer uso de algunos puertos bloqueados por nuestro proxy (varios en muchos casos) padre; o que tengamos que clonar un repositorio GIT con el último parche de algun servicio crítico de nuestra red (y los desarrolladores solo lo brinden por protocolo GIT, no HTTP o HTTPs). Son muchos los casos en que por la propia seguridad y actualización de sistemas debemos acceder por puertos diferentes al 80 o 443 y aquí es donde nos enfrentamos a los proxy padre muy restrictivos.

Me centraré en el cliente para Debian, pues es la distribución de Linux que utilizo; pero para los demás sistemas operativos es análogo, pues el fichero de configuración es común.

## Diferencia entre proxy HTTP y SOCKS

Seguro han visto esta diferenciación en alguna que otra configuración de los programas a la hora de definir el proxy, pues estos 2 tipos de proxy son muy diferentes:

### Proxy HTTP

Este tipo como su nombre lo indica, es pensado para la navegación web, pues en realidad lo que se hace es enviar la solicitud de la página a navegar (o acceder) al servidor proxy y éste es quien realiza la petición, cacheando o no la respuesta, para luego retornarla al cliente.

### Proxy SOCKS

Este tipo de proxy es más directo, en el sentido que, como su nombre lo indica, conecta directamente el cliente con el servidor, sirviendo simplemente de pasarela, es ampliamente utilizado para simular una conección directa. Por ejemplo, la aplicación `TOR` es un proxy SOCKS.

Este es de los proxy que menos se instalan en la infraestructura, puesto que el otro tipo (HTTP) es más enfocado a la navegación.

### Caso HTTPS

En el caso del protocolo HTTPS (generalmente puerto TCP 443), un proxy SOCKS no tiene problemas, puesto que comunica directamente los dos nodos (como se definía anteriormente); pero un proxy HTTP, al hacer de puente, no pudiera en teoría funcionar, puesto que tendría que recibir las peticiones para poder resolverlas, caso en que el HTTPS por ser un protocolo cifrado entre el servidor remoto y el cliente no lo permitiría sin un ataque de MITM.

Para este caso los proxy HTTP (y los navegadores o programas clientes) utilizan un método llamado **CONNECT** que no es más que pedirle una comunicación directa (tipo SOCKS) entre los 2 nodos. Una vez establecida la comunicación, es un enlace directo entre el cliente y el servidor, por lo que todo lo que se transmite entre los extremos es cifrado y ajeno al proxy.

## Manos a la obra

Para que OpenVPN se pueda comunicar a través de un proxy HTTP, éste debe permitir el método **CONNECT**, por lo que debemos tener un servidor remoto que esté escuchando por un puerto por el cuál el proxy permita la comunicación directa (como el TCP 443).

Hay muchos sitios en Internet que muestran listas de servidores de OpenVPN libres, como

 * [Lista 1](https://www.bestvpnserver.com/list-of-top-free-openvpn-servers/)
 * [Lista 2](http://www.vpngate.net/en/)

solo bastaría descargar su fichero de configuración y debajo de `remote vpn_hostname 443 tcp` adicionarle:

```
http-proxy 1.2.3.4 8080
http-proxy-retry
```

en caso de necesidad de autenticación:

```
http-proxy 1.2.3.4 8080 auto
<http-proxy-user-pass>
username
password
</http-proxy-user-pass>
http-proxy-retry
```

De esta manera solo quedaría ejecutar nuestro cliente OpenVPN con la configuración

```terminal
$ openvpn config.ovpn
```
