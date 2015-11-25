---
layout: post
title: ARP Spoofing
tags: [arp, network, spoofing]
summary: Algunos conceptos acerca del ARP Spoofing
---

## Introducción

Hoy en día el uso de las tecnología ha tenido un auge importantísimo en el día a día nuestro, automatizándonos y por ende facilitándonos la mayoría de las recurrentes tareas que realizamos. En nuestro país cada vez más se ve inundada por nuevos elementos, ya sean de software como de hardware y vamos sin querer haciendo a un lado un aspecto muy importante dentro de ellos... la seguridad. Hoy la mayoría de nosotros subvaloramos la seguridad que determinado dispositivo debe tener, por lo que no nos preocupamos mucho de esto y por tanto dejamos siempre todas las configuraciones por defecto de como viene, ejemplo? instalamos un servidor de base de datos y no cambiamos la contraseña maestra, instalamos un *router* inalámbrico en la casa y no lo configuramos con una contraseña ni con filtro para permitir que solo el dueño sea el que lo administre, entre otros elementos.

En este blog comentaré acerca de todo tipo de elementos de la tecnología, en este caso comentaré acerca de algunos elementos de seguridad, principalmente hablaré acerca de una técnica utilizada para *"mirar"* (analizar en buenos términos) toda la información que circula entre dos dispositivos. Aclaro que lo que aquí expreso, aunque nunca diré explícitamente como se realizan estos ataques, es solo para que las personas tengan el conocimiento de cómo funciona y por ende como protegerse de estas técnicas, no me responsabilizo de la falta de seguridad de sistemas donde estos ataques puedan ser cometidos por los conocimientos aquí adquiridos.

## ARP Spoof

Nombrando el ataque que hoy comentaré, se trata del *"ARP Spoof"* o envenenamiento por ARP, existen disímiles técnicas de "envenenamiento", como DHCP, ICMP, DNS, etc. Primeramente decir que ARP (Protocolo de resolución de nombres o *Address Resolution Protocol*) es un protocolo encargado de buscar en la red la correspondencia de una dirección de hadware (las famosas MAC) con respecto a su dirección IP. Para un ejemplo más práctico hagamos un "ping" a un ip cualquiera, en este ejemplo al 192.168.0.15:

{% highlight bash linenos %}
ping 192.168.0.15
{% endhighlight %}

que es lo que sucede a manera de resumen y teniendo en cuenta el ARP (evitando muchos elementos que trabajan en su conjunto). Los sistemas operativos que conozco (Microsoft(R) Windows(TM), Apple(R) MacOS(R) o GNU/Linux) cuentan, para un óptimo trabajo con los paquetes ARP, con una tabla interna que relaciona los IP con las MAC, de esta manera no tienen que buscar una MAC si ya las tienen. Bien, volviendo al comando "ping", el paquete de ICMP (el paquete que se envía realmente con el "ping") no se direcciona por el IP, sino por la MAC de destino, y como se conoce esa MAC? aquí es donde entra el ARP, si el SO (Sistema Operativo) no contiene esa relación dentro de su "tabla arp" (la cual se puede ver con "arp -a") entonces envía al broadcast (toda la red) un paquete ARP especial que contiene el IP que requiere, cuando ese paquete llega a la PC directamente o través de varios routers, este responde con su dirección de MAC con un paquete de respuesta de ARP. Cuando éste último arriba a la PC entonces el SO ubica una nueva entrada en la tabla arp y comienza a comunicarse directamente con esa MAC. No es solamente el comando "ping" el que realiza esta tarea, sino todos los programas que requieran del protocolo IP para su comunicación, de hecho la mayoría de ellos.
Bien una vez que la PC tiene la MAC correspondiente a una PC remota, cada vez que se requiera comunicar con ella, no realiza todo el proceso de pedido de MAC a través de su IP, sino que consulta en su tabla arp y se comunica directamente; claro, cada cierto tiempo se vuelve a pedir la MAC, para casos de cambios de IP, etc. Ahora, hablando más concretamente del ataque de "ARP Spoof", supongamos que tenemos el siguiente escenario:

![Escenario]({{ site.url }}/images/posts/arp_spoofing/escenario.png)

Una PC que se conecta a un servidor utilizando para utilizar un servicio determinado, dígase XMPP (Jabber, chat), HTTP, IMAP (Correo), POP3 (Correo), etc. Además existe una persona (Atacante), la cual, en este caso se encuentra en la misma red que usted y quiere espiar dicha conexión entre el servidor y el cliente. Aquí también hay que dar un ligero detalle acerca del dispositivo que contiene todas las conexiones, ya sea un "Switch" o un HUB, la principal diferencia entre estos es que el "Switch" es orientado a conexiones directas, es decir lo que envíe una PC hacia otra, solo se transmite por los puertos requeridos para dicha conexión, mientras que el HUB lo replica todo; existen muchas diferencias adicionales, pero no es objetivo de este post. Para nuestro caso, estamos en una red interna administrada por un "switch", por lo que los paquetes entre el cliente y el servidor no llegan a nuestra PC.
Adentrándonos en el ataque como tal, consiste en que el atacante envía un paquete de respuesta ARP hacia ambas direcciones, dígase el cliente y el servidor, informando acerca de una nueva dirección MAC para un IP determinado (en caso del cliente, informando que el IP del servidor tiene otra MAC y para el caso del servidor que el IP del cliente se encuentra en otra MAC), dicha MAC nueva corresponde a la dirección física del atacante, estos paquetes se mantienen emitiendo desde la PC del atacante por intervalos de tiempos cortos, para de una manera forzar al SO de que cambie su tabla arp. Habiendo logrado esto, el escenario se transformaría de la siguiente forma:

![Escenario ARP](/images/posts/arp_spoofing/escenario_arp.png)

En este caso quedaría que a manera de ver del cliente se está comunicando correctamente con el IP del servidor, igualmente para el servidor con el IP del cliente, pero a nivel de paquetes, toda la comunicación se realiza a través del equipo del atacante. Esto puede volcarse en disímiles sub-ataques, díganse denegación de servicio (DoS), extración de usuarios y contraseñas, suplantación de sitios Web, etc.
Mostraré a través de capturas de pantalla lo que sucede en el ataque, un ejemplo antes de aplicar el ataque y luego de realizarlo, en este caso el cliente es una PC Virtual y el servidor es la puerta de enlace, el atacante, yo.

![Antes del ataque]({{ site.url }}/images/posts/arp_spoofing/antes_ataque.png)

Aquí se evidencia viendo la tabla arp en el cliente cual es la dirección de .254, la puerta de enlace en este caso, el ip del atacante es el .168 el cual tiene un mac determinado. A la hora de realizar el ataque quedaría:

![Despues del ataque]({{ site.url }}/images/posts/arp_spoofing/despues_ataque.png)

Ahora podemos observar como en la tabla de arp la mac de la puerta de enlace (.254) es igual a la del atacante (.168).

Ahora bien, ya se entiende como funciona dicho ataque, como protegernos es lo importante. Bien, la manera más eficaz de determinar cuando nos encontramos ante un ataque de este tipo es verificar nuestra tabla arp utilizando los comandos definidos por el SO:

{% highlight bash linenos %}
arp -a
{% endhighlight %}

de esta manera revisaremos de forma manual en cuanto a que todas las MAC de las PCs deben ser diferentes, prestando principal atención cuando nos encontramos en redes que cuentan con puertas de enlace a que su MAC se encuentre "clonada" por otra PC. Existen herramientas para automatizar dichos procesos, como lo son los Firewalls, además de otros especializados en este campo, como "arpwatch", "arpalert", etc.

De manera general las maneras de defendernos serían:

1. Establecer de una manera determinada por el SO o por el dispositivo hardware las tablas ARP de manera manual y eliminar la posibilidad de que se actualicen automáticamente, así se evitan los paquetes autogenerados por el atacante para confundir las tablas ARP. La desventaja de este método es que para redes medianamente grandes es una ardua tarea, pues se tendría que configurar todos los puntos de red para cada actualización de IP nueva, o la inclusión de nuevos dispositivos.
2. En cuanto a GNU/Linux se puede utilizar una opción del kernel que impide que sean aceptadas los pedidos de ARP "gratuitos", es decir que lleguen respuestas de paquetes ARP si haberlo solicitado. Importante aclarar que esta opción por defecto viene activada, pero para comprobación de los lectores pueden verlo con el comando en la consola:

{% highlight bash linenos %}
cat /proc/sys/net/ipv4/conf/all/arp_accept
{% endhighlight %}

En caso de que el resultado no sea "0" es un riesgo de seguridad y deberían ponerlo inmediatamente a 1, de esta manera y desde una terminal con acceso de administración "root":

{% highlight bash linenos %}
echo "0" > /proc/sys/net/ipv4/conf/all/arp_accept
{% endhighlight %}

En Microsoft(R) Windows(TM) esto lo logramos con el propio firewall que trae por defecto. Muy importante aclarar que esto evita solamente aquellos paquetes respuesta de ARP que no hayan sido solicitados, pero si el atacante a la hora de analizar un pedido que se haya realizado y responde a dicho pedido, no se bloqueará, pues se tomará como una respuesta esperada.

## Ver:

- [Ecured](http://www.ecured.cu/index.php/Arp)
- [ARP-Poisoning](http://www.arppoisoning.com/best-practices-for-defending-against-arp-poisoning-2/)
- [Raymond](https://www.raymond.cc/blog/protect-your-computer-against-arp-poison-attack-netcut/)
