---
layout: post
title: WireGuard, un ejemplo práctico
tags: [vpn, wireguard, linux, seguridad]
summary: WireGuard usado en un caso real
---

## Introducción

En un [artículo anterior]({{ site.url }}2018/04/21/wireguard_server.html) comentaba sobre las bondades de WireGuard, así como su instalación y configuración básica. Algunos pudieran ver ese artículo y preguntarse sobre cómo usarlo realmente, no solo basta con saber el cómo, también necesitamos ver el dónde.

En este caso explicaré un ejemplo práctico para implementar WireGuard en su red. A grandes rasgos utilizaríamos a WireGuard para proteger todas las comunicaciones entre los clientes (usuarios de la red) y los servidores; además de una instancia adicional de WireGuard que le permitirá al encargado de Seguridad Informática acceder a los sistemas de monitoreo por el mismo canal de los usuarios normales, sin comprometer la seguridad de dichos servicios. Quedaría algo así:

![Escenario]({{ site.url }}images/posts/wireguard_server/example.png)

Como podemos apreciar en la imagen anterior, tendríamos un firewall entre los usuarios de la red y los servidores. Dicho firewall brindaría los servicios de WireGuard a los usuarios, los cuales están conectados por el canal físico hacia éste. Al WireGuard crear interfaces virtuales TUN, el firewall solo permitiría la comunicación hacia los servicios a través de estas interfaces virtuales nuevas (las creadas por WireGuard), denegando toda comunicación restante. De esta manera todo dispositivo que no se encuentre como cliente del WireGuard no podrá consumir de los servicios, ni podrá ver la comunicación, debido a la encriptación brindada por WireGuard.

Si se fijan, se evidencia el ejemplo de la utilización de otro canal de WireGuard específico para los especialistas de Seguridad Informática.

## Manos a la obra

Primero debemos en nuestro firewall levantar los dos servidores de WireGuard (asumo que hayan leído el [artículo anterior]({{ site.url }}2018/04/21/wireguard_server.html)), uno para los usuarios normales y otro para el encargado de la Seguridad Informática.

```terminal
# Generando las llaves privadas de los servidores

$ wg genkey
YMbajbQrpvYHgFaMgUSYuwMBa9wnLa4JKG+lWMwrmGY=

$ wg genkey
kHv5yTgIvGzrbwf/VydqDHvEH4//88D/TDF/8i8gxF4=
```

Luego crearíamos con algun editor de texto los ficheros `/etc/wireguard/empresa.conf` y `/etc/wireguard/si.conf` y copiaremos la siguiente configuración:

```
# empresa.conf
# Definición de la Interfaz
[Interface]

# Establecer el IP de la interfaz con la máscara de red
Address = 172.16.0.1/24

# Puerto UDP de escucha del servidor
ListenPort = 51820

# Llave privada generada anteriormente
PrivateKey = YMbajbQrpvYHgFaMgUSYuwMBa9wnLa4JKG+lWMwrmGY=

# Indica que cada configuración establecida a través de la interfaz de
# WireGuard sea salvada en el fichero una vez detenido el servidor
SaveConfig = false
```

```
# si.conf
# Definición de la Interfaz
[Interface]

# Establecer el IP de la interfaz con la máscara de red
Address = 172.16.1.1/24

# Puerto UDP de escucha del servidor
ListenPort = 51821

# Llave privada generada anteriormente
PrivateKey = kHv5yTgIvGzrbwf/VydqDHvEH4//88D/TDF/8i8gxF4=

# Indica que cada configuración establecida a través de la interfaz de
# WireGuard sea salvada en el fichero una vez detenido el servidor
SaveConfig = false
```

Ahora que tenemos las configuraciones solo falta habilitar y arrancar los servidores de WireGuard (para eso utilizaremos la ayuda de `systemd` y `wg-quick`):

```terminal
# empresa.conf
$ systemctl enable wg-quick@empresa
$ systemctl start wg-quick@empresa

# si.conf
$ systemctl enable wg-quick@si
$ systemctl start wg-quick@si
```

Podemos asegurarnos de que todo está correcto verificando que se hayan creado 2 interfaces nuevas con el nombre del fichero de configuración, ejemplo:

```terminal
$ ip a
...
5: empresa: <POINTOPOINT,NOARP,UP,LOWER_UP> mtu 1420 qdisc noqueue state UNKNOWN group default qlen 1000
    link/none
    inet 172.16.0.1/24 scope global empresa
       valid_lft forever preferred_lft forever
6: si: <POINTOPOINT,NOARP,UP,LOWER_UP> mtu 1340 qdisc noqueue state UNKNOWN group default qlen 1000
    link/none
    inet 172.16.1.1/24 scope global si
       valid_lft forever preferred_lft forever
...
```

Ya con esto tenemos los servidores de WireGuard listos, solo falta agregar los clientes.

## Agregando clientes

Cada cliente que se requiera conectar al servidor tiene que tener su entrada correspondiente en el fichero de configuración de dicho servidor WireGuard; para ello se necesita un par de llaves pública y privada, donde la privada la tiene el cliente y el servidor solo necesita definir la pública dentro de su configuración. En este caso existen 2 variantes:

 * Generar el par de llaves en cada cliente y copiar la pública en el servidor.
 * Generar el par de llaves de cada cliente en el propio servidor.

La diferencia entre ellas es el nivel de seguridad (y paranoia) que queremos, puesto que lo más seguro es generarlo en cada cliente por separado para aumentar el principio de insertidumbre en la generación de comportamientos aleatorios de la generación de las llaves. Pero es mucho más cómodo realizarlo en el propio servidor, por lo que es a gusto del lector como lo desea hacer.

>### En Debian
>
>```terminal
># Se genera la privada
>$ wg genkey
>YKB7m15Art0Y9jcAYr/9DX6mr6cr28uUzABI7Q1TwH0=
>
># Se genera la pública a partir de la privada
>$ echo -n "YKB7m15Art0Y9jcAYr/9DX6mr6cr28uUzABI7Q1TwH0=" | wg pubkey
>```

> ### En Windows
>
> Utilizando el cliente TunSafe
>
> ![Opciones]({{ site.url }}/images/posts/wireguard_server/tunsafe_options_button_select.png)
>
> ![Menú de Opciones]({{ site.url }}/images/posts/wireguard_server/tunsafe_option_generate_key_pair_select.png)
>
> ![Generación de llaves]({{ site.url }}/images/posts/wireguard_server/tunsafe_option_generate_key_pair.png)


Una vez que se tenga el par de llaves, se toma la pública y agregamos el siguiente fragmento a la configuración correspondiente, por ejemplo en `/etc/wireguard/empresa.conf`:

```
# empresa.conf
# Por cada cliente se establece una sección como esta
[Peer]

# Llave pública del cliente
PublicKey = JP55vfgoBIRbXuasBD0Gn5L6mb8KsVwdZiOfdi/cfCs=

# IPs (con máscara) que puede tener el cliente que se conecte con esta llave,
# pudiera usarse por ejemplo para un cliente que se conecte por 2 VPN
AllowedIPs = 172.16.0.2/32
```

Una vez agregados los clientes, procedemos y reiniciamos el servidor correspondiente:

```terminal
$ systemctl restart wg-quick@empresa

```

> WireGuard soporta la configuración directa en cuanto está levantado el servidor, incluso agregar nuevos clientes y que se escriba en el fichero de configuración una vez terminado el proceso, éste método no lo explico por ser un poco más complicado; además en mi opinión le veo un defecto y es que si tenemos un reinicio inesperado, toda la configuración "en caliente" que hayamos hecho se habrá perdido.

## Configurando los clientes

### Debian

Para configurar el cliente en Debian el procedimiento es parecido al servidor (explicado igualmente en el artículo anterior sobre WireGuard), solo basta con agregar un fichero de configuración en `/etc/wireguard/empresa.conf` con el siguiente contenido:

```
# Definición de la Interfaz
[Interface]

# IP del cliente que tiene que coincidir con el permitido en el servidor
# para este cliente
Address = 172.16.0.2/32

# Llave privada generada
PrivateKey = YKB7m15Art0Y9jcAYr/9DX6mr6cr28uUzABI7Q1TwH0=

# Si queremos establecer un DNS
DNS = 172.16.0.1

# Definición del servidor al cual conectarse
[Peer]

# Se define la llave pública de dicho servidor
PublicKey = YMbajbQrpvYHgFaMgUSYuwMBa9wnLa4JKG+lWMwrmGY=

# En este apartado se define los IPs que enrutarán por esta interfaz,
# como se puede ver en este caso son todas, es decir, sería la puerta
# de enlace por defecto, todas las conecciones externas se enrutan hacia
# el servidor de WireGuard
AllowedIPs = 0.0.0.0/0

# Dirección del servidor de WireGuard
Endpoint = 10.0.0.1:51820

# Esto define un intervalo en segundos que WireGuard envía un paquete
# nulo para "mantener viva" la conección. Esto es útil en casos de NAT
# para que el firewall mantenga el mismo IP asociado a la conección.
# Comento por defecto debido a que en nuestro caso no es NAT
# PersistentKeepalive = 25
```

Arrancamos el cliente de la misma manera que el servidor:

```terminal
$ systemctl enable wg-quick@empresa
$ systemctl start wg-quick@empresa
```

### Windows

En este caso utilizaremos el cliente [TurnSafe](https://tunsafe.com/download)

![Opción de configurar]({{ site.url }}/images/posts/wireguard_server/tunsafe_edit_config_button_select.png)

Se abre el editor por defecto para configurar el cliente (o servidor) de WireGuard

![Editor de la configuración]({{ site.url }}/images/posts/wireguard_server/tunsafe_notepad_config_edit.png)

Como pueden observar la configuración es idéntica a la de Debian.


Solo queda por último las reglas de firewall para el correcto enrutado de los paquetes que vienen solo por las interfaces virtuales, pero este proceso queda fuera del ejemplo debido a la gran variedad de vías por la cual hacerlo.
