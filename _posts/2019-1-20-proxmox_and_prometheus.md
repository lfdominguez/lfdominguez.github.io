---
layout: post
title: Monitorizando las máquinas virtuales de proxmox con Prometheus
tags: [métricas, monitoreo, linux, servidores, docker, grafana, node-exporter]
summary: Como instalar y configurar un *exporter* de Prometheus para obtener las métricas de las máquinas virtuales de Proxmox.
---

## Introducción

Muchos contamos con una infraestructura soportada sobre la plataforma de Proxmox y una de las necesidades que debemos tener es medir, métricamente, los diferentes aspectos de cada una de sus máquinas virtuales, ya sea LXC o KVM.

Algunos enfoques definen que se debe tener un programa colector en cada máquina virtual, pero el enfoque que traigo actualmente es el de usar un colector por nodo físico que hace uso del API de Proxmox para pedir las métricas de los nodos virtuales. En el caso de que sea un clúster, solo se necesita en un nodo, puesto que el API de Proxmox te da la información de todos los miembros del clúster.

### Instalando el *exporter*

Podemos encontrar el exporter en 