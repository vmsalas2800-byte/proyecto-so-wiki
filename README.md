# Wiki del Equipo - Wiki.js + PostgreSQL en Podman Pod

Este proyecto consiste en el despliegue de una plataforma de documentación basada en **Wiki.js**, utilizando **PostgreSQL 15-alpine** como motor de base de datos. El despliegue se realiza bajo una arquitectura de contenedores **rootless** y **daemonless** utilizando **Podman** en una instancia AWS EC2 [5, 6].

## Integrantes - Grupo 8
* Ángel Guillermo Huaman Cardenas
* Alvaro Montero Chacon
* Paul Alejandro Villena Romero
* Victor Manuel Salas Medina

## Arquitectura
El despliegue utiliza un **Podman Pod** denominado "pod-wiki", permitiendo que la aplicación y la base de datos compartan el mismo **Network Namespace** y se comuniquen de forma segura a través de "localhost".

### Especificaciones Técnicas:
- **SO Host:** Ubuntu Server 24.04 LTS (AWS EC2 t3.micro).
- **Límites de Recursos (Cgroups v2):** La base de datos está limitada a **0.5 CPU** y **256MB de RAM**.
- **Acceso Externo:** Dominio dinámico vía DuckDNS: "http://wikii-esan-victorr.duckdns.org".

## Instrucciones de Despliegue
### Requisitos Previos
1. Instalar Podman: "sudo apt update && sudo apt install -y podman".
2. Configurar puertos privilegiados para el puerto 80: "sudo sysctl net.ipv4.ip_unprivileged_port_start=80".
