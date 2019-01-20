---
layout: post
title: Servidor de métricas Prometheus
tags: [métricas, monitoreo, linux, servidores, docker]
summary: Como instalar y configurar Prometheus, un servidor de métricas para nuestra infraestructura
---

## Introducción

Prometheus es un sistema de monitoreo que a diferencia de sus homólogos utiliza un enfoque diferente, en lugar de que los servicios que lo soporten o los programas que obtienen las métricas se las envíen a Prometheus, éste es quien se conecta a ellos cada cierto tiempo y recopila toda la información.
Las ventajas y desventajas de esta manera de trabajar pueden profundizarla más en [el sitio oficial](https://prometheus.io/docs/introduction/faq/#why-do-you-pull-rather-than-push) o en [su blog](https://prometheus.io/blog/2016/07/23/pull-does-not-scale-or-does-it/).
Está programado en Go, por lo que su instalación es sumamente sencilla y no tiene dependencia alguna, es un simple ejecutable.

## Instalación genérica



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
$ docker run --name prometheus -d -p 9090:9090 prometheus/prometheus
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