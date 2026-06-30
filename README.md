# Wiki del Equipo - Wiki.js + PostgreSQL en Podman Pod

Este proyecto consiste en el despliegue de una plataforma de documentación basada en **Wiki.js**, utilizando **PostgreSQL 15-alpine** como motor de base de datos. El despliegue se realiza bajo una arquitectura de contenedores **rootless** y **daemonless** utilizando **Podman** en una instancia AWS EC2.

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
1. Instalar Podman: sudo apt update && sudo apt install -y podman
2. Configurar puertos privilegiados para el puerto 80: sudo sysctl net.ipv4.ip_unprivileged_port_start=80
3. Persistencia tras reinicio del sistema: echo "net.ipv4.ip_unprivileged_port_start=80" | sudo tee -a /etc/sysctl.conf
   
## Configuracion de nombres cortos
Se editó el archivo de registros para permitir el uso de nombres de imagen simplificados como postgres en lugar de URLs largas:

1. Comando: sudo nano /etc/containers/registries.conf
2. Configuración: unqualified-search-registries = ["docker.io", "quay.io"]
3. Verificación: cat /etc/containers/registries.conf 

Paso 1: Creación del Pod:
Creamos el espacio lógico con mapeo del puerto 80 (host) al 3000 (interno): podman pod create --name pod-wiki -p 80:3000

Paso 2: Despliegue de la Base de Datos (PostgreSQL):
Lanzamos el contenedor con límites estrictos gestionados por el kernel:

podman run -d --name db-wiki --pod pod-wiki \
  --cpus="0.5" --memory="256m" \
  -e POSTGRES_DB=wiki \
  -e POSTGRES_USER=wiki_user \
  -e POSTGRES_PASSWORD=contrasena_esan \
  postgres:15-alpine

Paso 3: Despliegue de la Aplicación (Wiki.js):
Conexión directa por la red interna del Pod ("localhost"):

podman run -d --name app-wiki --pod pod-wiki \
  -e DB_TYPE=postgres -e DB_HOST=localhost -e DB_PORT=5432 \
  -e DB_USER=wiki_user -e DB_PASS=contrasena_esan -e DB_NAME=wiki \
  ghcr.io/requarks/wiki:2

Instrumentación del Sistema Operativo
Para validar el comportamiento del kernel bajo los contenedores, se ejecutaron:

1.  **Medición de Cgroups:** Verificación de límites en tiempo real: podman stats --no-stream
2.  **Aislamiento de Procesos (PID Namespaces):** Observación de la jerarquía *fork/exec*: podman top db-wiki
3.  **Inspección del Kernel (/proc):** Identificación del PID real en el host y su mapa de memoria virtual:
    
   PID=$(podman inspect db-wiki --format '{{.State.Pid}}')
   cat /proc/$PID/maps | head -n 10
    

## Resultados del Experimento de Carga
Se realizó una prueba de carga concurrente para observar la respuesta de los recursos.
*   **CPU app-wiki:** Incrementó de **1.60%** a **31.03%** bajo carga, estabilizándose inmediatamente después, lo que demuestra la eficiencia en la gestión de procesos del kernel.
*   **Memoria db-wiki:** Se mantuvo estrictamente bajo el límite de **256MB**, protegiendo la estabilidad del host AWS ante picos de consumo.

## Conclusiones
*   El modelo **rootless y daemonless** de Podman minimiza significativamente la superficie de ataque al gestionar contenedores como procesos hijos del usuario.
*   La arquitectura de **Pods** optimiza sistemas distribuidos al permitir comunicación por "localhost", eliminando la latencia de redes virtuales complejas.
*   La instrumentación mediante **cgroups v2** es vital para evitar que fallos en una aplicación agoten los recursos físicos del servidor host.

## Enlaces
*   **Repositorio:** [https://github.com/vmsalas2800-byte/proyecto-so-wiki](https://github.com/vmsalas2800-byte/proyecto-so-wiki)
*   **Video Demo:** [Enlace pendiente]
