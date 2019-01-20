---
layout: post
title: Servidor de métricas Prometheus y aplicación práctica con Node-Exporter y Grafana
tags: [métricas, monitoreo, linux, servidores, docker, grafana, node-exporter]
summary: Como instalar y configurar Prometheus, un servidor de métricas para nuestra infraestructura, así como node-exporter y visualizar con Grafana.
---

## Introducción

Prometheus es un sistema de monitoreo que a diferencia de sus homólogos utiliza un enfoque diferente, en lugar de que los servicios que lo soporten o los programas que obtienen las métricas se las envíen a Prometheus, éste es quien se conecta a ellos cada cierto tiempo y recopila toda la información.
Las ventajas y desventajas de esta manera de trabajar pueden profundizarla más en [el sitio oficial](https://prometheus.io/docs/introduction/faq/#why-do-you-pull-rather-than-push) o en [su blog](https://prometheus.io/blog/2016/07/23/pull-does-not-scale-or-does-it/).
Está programado en Go, por lo que su instalación es sumamente sencilla y no tiene dependencia alguna, es un simple ejecutable.

Vamos a ver un ejemplo de aplicación con Node-Exporter y Grafana para levantar una mínima infraestructura de monitoreo en nuestra red.

## Instalación genérica

Como comentaba anteriormente, debido a su lenguaje de programación, es simplemente ir a la página de [GitHub](https://github.com/prometheus/prometheus/releases/latest) y descargar la versión estable para el sistema operativo que querramos (incluido windows ;) ). Una vez descomprimido, es simplemente ejecutarlo como un programa más, queda fuera del ámbito del manual la creación de un servicio que lo use.

Alguno de sus argumentos básicos son:

 * `--config.file` Ubicación del fichero de configuración en `YML`. Por defecto es `/etc/prometheus/prometheus.yml`.
 * `--web.listen-address` Dirección por donde escuchará la UI, el API y la telemetría. Por defecto es `0.0.0.0:9090`.
 * `--storage.tsdb.retention` Tiempo de retención de los datos de las métricas (Prometheus no está pensado para guardar un histórico largo de datos, para ello hay soluciones que se integran a Prometheus). Por defecto es `15d`.
 * `--log.level` Nivel de bitácora. Por defecto es `info`.

## Instalando en Debian

En el repositorio de Debian se encuentran los paquetes referentes a Prometheus. En mi caso uso siempre en mis servidores la versión estable de éste (que en el momento de escribir es *stretch*).
La versión que se encuentra en los repositorios estables es la 1.x, pero Prometheus ha cambiado su modelo de almacenamiento completamente en la versión 2.x, por lo que haría incompatible los 2 modelos. En este caso tendremos que utilizar el repositorio *back-ports* de Debian, que no es más que ciertos programas que han sido portados para la versión estable de Debian.

 1. Primero configuramos el repositorio para agregar al `sources.list` la nueva fuente:

 ```terminal
 $ echo "deb http://ftp.debian.org/debian stable-backports main" >> /etc/apt/sources.list
 ```
 
 2. Luego actualizamos los índices
 
 ```terminal
 $ apt update
 ```

 3. Y ahora instalamos Prometheus, indicando que queremos la de *back-ports*

 ```terminal
 $ apt install prometheus/stable-backports
 ó
 $ apt -t stable-backports install prometheus
 ```

## Ejecutando desde Docker

Como siempre ejecutarlo desde Docker es más fácil, la imagen oficial de Prometheus la podemos encontrar tanto en [Quay.io](https://quay.io/repository/prometheus/prometheus) como en [Docker Hub](https://hub.docker.com/r/prom/prometheus/):

### Quay.io
```terminal
$ docker run --name prometheus -d -p 9090:9090 quay.io/prometheus/prometheus
```

### Docker Hub
```terminal
$ docker run --name prometheus -d -p 9090:9090 prom/prometheus
```

### Opciones

 * `--name` Le asignamos el nombre `prometheus` a esa instancia, de manera que es más fácil encontrarla, por ejemplo, con un `docker ps`
 * `-d` Le decimos a Docker que lo inicie en modo *demonio*, de esta manera se devuelve el control a la terminal y la instancia se queda ejecutándose en el fondo.
 * `-p` Definimos que exporte el puerto `9090` de la instancia, hacia el `9090` de nuestra PC (o servidor) en [http://localhost:9090](http://localhost:9090).

## Verificando instalación

Ahora podremos acceder por HTTP hacia el servidor Prometheus usando un navegador. En esa interfaz podremos hacer consultas para probar las gráficas y los datos, así como ver el estado de los servicios a los cuales Prometheus se conecta para obtener la información de las métricas.

### Página de consultas

![Escenario]({{ site.url }}images/posts/prometheus/web_graph.webp)

### Página de chequeo de los clientes

![Escenario]({{ site.url }}images/posts/prometheus/web_target_status.webp)

## Breves sobre la Configuración

El fichero de configuración de Prometheus por lo general se encuentra en `/etc/prometheus/prometheus.yml` y se explica por sí solo. Algunas precisiones:

 * `scrape_interval` Es el intervalo en que consulta cada servicio para obtener las métricas.
 * `evaluation_interval` Es el intervalo en que evalúa las reglas de alerta (no tratadas en el presente post).
 * `scrape_configs` Es donde va la configuración de los servicios a consultar:

 Partiendo un ejemplo simple, funcional y que se explica por si solo, es simplemente ubicarlo dentro de `scrape_configs`:

 ```yml
 - job_name: 'squid'
    static_configs:
      - targets:
        - 192.168.0.128:9399
    metrics_path: /metrics
 ```

## PromQL

Para terminar hablo sobre *PromQL* que es el lenguaje de consultas a Prometheus. Permite seleccionar y realizar operaciones sobre las series de tiempo. Puedes conocer más sobre el lenguaje en su [sitio oficial](https://prometheus.io/docs/prometheus/latest/querying/basics/). Solo mostraré algunos ejemplos rápidos (tomados de su [sitio oficial](https://prometheus.io/docs/prometheus/latest/querying/examples/)):

### Operaciones simples

 * Retornar todas las series de tiempo de la métrica `http_requests_total`:
 
 ```promql
 http_requests_total
 ```

 * Retornar todas las series de tiempo de la métrica `http_requests_total`, pero especificando algunas etiquetas:

 ```promql
 http_requests_total{job="apiserver", handler="/api/comments"}
 ```

 * De la anterior, retornar solamente un rango de tiempo (en este caso 5 min):

 ```promql
 http_requests_total{job="apiserver", handler="/api/comments"}[5m]
 ```

 * La búsqueda en las etiquetas se puede hacer por expresiones regulares, como subcadenas a buscar, por ejemplo, para sacar todos los datos referentes a la métrica donde el `job` termine en `server`:

 ```promql
 http_requests_total{job=~".*server"}
 ```

 Todas las expresiones regulares en Prometheus utilizan la sintaxis [R2](https://github.com/google/re2/wiki/Syntax).

 Por ejemplo para excluir utilizando las etiquetas, seleccionando todo menos los que tengas `status` diferente a `4xx`:

 ```promql
 http_requests_total{status!~"4.."}
 ```

### Usando funciones y operadores

 * Retornar el ritmo(*rate*) de la métrica `http_requests_total` durante 5 min:

 ```promql
 rate(http_requests_total[5m])
 ```

 Ahora, suponiendo que tenemos una etiqueta que define el `job` podemos sumar todos los *rate* y agruparlos por el `job`:

 ```promql
 sum(rate(http_requests_total[5m])) by (job)
 ```

 * Si tenemos métricas que tienen la misma definición de etiquetas podremos realizar cálculos matemáticos con ellas, por ejemplo, para calcular la memoria sin usar en MBytes por una instancia de un nodo ficticio que tendría las métricas `instance_memory_limit_bytes` y `instance_memory_usage_bytes`:

 ```promql
 (instance_memory_limit_bytes - instance_memory_usage_bytes) / 1024 / 1024
 ```

 * Lo mismo que lo anterior pero ahora sumado por aplicación y proceso:

 ```promql
 sum(
  instance_memory_limit_bytes - instance_memory_usage_bytes
 ) by (app, proc) / 1024 / 1024
 ```

## Node-Exporter

Es un *exporter* de Prometheus (así son llamados los programas encargados de obtener las métricas y brindarlas en un formato que Prometheus pueda obtener, algunos programas ya la traen incluidas, en otros casos se debe usar uno externo). Con Node-Exporter, Prometheus puede consumir, entre muchas otras, las siguientes métricas:

 * `arp` Expone las estadísticas ARP de `/proc/net/arp`.
 * `cpu`
 * `diskstats` Expone las métricas de I/O de los discos.
 * `entropy` Expone la entropía disponible (muy importante para la criptografía).
 * `filesystem` Expone las estadísticas de los sistemas de ficheros, como el espacio en disco usado, etc.
 * `hwmon` Expone los sensores de hardware de `/sys/class/hwmon/`
 * `loadavg` Expone el promedio de carga del nodo.
 * `meminfo` Expone las estadísticas de la RAM.
 * `netclass` Expone la información de las interfaces de red de `/sys/class/net/`
 * `netdev` Más estadísticas de red, como los *bytes* transferidos, etc.
 * Entre muchas otras ...

### Instalación genérica

Podemos seguir el mismo procedimiento de Prometheus, pero buscando la versión estable [aquí](https://github.com/prometheus/node_exporter/releases/latest).

### Instalación en Debian

Debian en su repositorio cuenta con el Node-Exporter, el paquete se llama `prometheus-node-exporter`, por lo que con `apt`:

```terminal
$ apt install prometheus-node-exporter
```

  En caso de que se quiera usar en la versión estable de Debian, hay que activar el repositorio de *backports* como mismo se realizó en la instalación de Prometheus.

### Ejecutando desde Docker

Podemos usar la imagen de Quay.io:

```terminal
docker run -d \
  --net="host" \
  --pid="host" \
  -v "/:/host:ro,rslave" \
  quay.io/prometheus/node-exporter \
  --path.rootfs /host
```

Lo más relevante es que del host se expone la raíz, para que Node-Exporter pueda obtener las métricas y usa la propia interfaz de red del nodo, para evitar medir solo la interfaz virtual.

Para más información ir a [su sitio](https://github.com/prometheus/node_exporter)

### Configurando Prometheus

Node-Exporter por defecto usa el puerto `9100` y `/metrics` para exportar las métricas, por lo que vamos a la configuración de prometheus `/etc/prometheus/prometheus.yml` y agregamos el (o los) nodo(s) que exponen sus métricas por debajo de `scrape_configs`:

```yml
- job_name: 'node_exporter'
static_configs:
  - targets:
    - ip_node_1:9100
    - ip_node_2:9100
    - ip_node_3:9100
    - ip_node_4:9100
metrics_path: /metrics
```

En mi caso lo tengo instalado en cada servidor Proxmox.

## Grafana

Ahora toca visualizar todo. Uso Grafana por sus potencialidades y se ha convertido en la plataforma de visualización de facto en mi entorno, [acá](https://raw.githubusercontent.com/rfrail3/grafana-dashboards/master/prometheus/node-exporter-full.json) les dejo un *dashboard* específico para Node-Exporter, con una foto para que vean como queda:

![Grafana]({{ site.url }}images/posts/prometheus/grafana_1.webp)

![Grafana]({{ site.url }}images/posts/prometheus/grafana_2.webp)

![Grafana]({{ site.url }}images/posts/prometheus/grafana_3.webp)