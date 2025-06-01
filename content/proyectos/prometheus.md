---
title: "Monitoreo con Prometheus, Grafana y Nextcloud"
date: 2025-05-31
tags: ["docker", "prometheus", "grafana", "monitoring"]
description: "Escenario completo de monitoreo con servicios contenedorizados y visualización en tiempo real"
---

## Introducción

Este proyecto configura un entorno de monitoreo usando Prometheus y Grafana, además de un servidor Nextcloud. Todo corre en contenedores Docker.

## Pasos

```bash
# Crear carpeta del proyecto
mkdir monitoring-stack && cd monitoring-stack

```

```bash
# Crear carpeta del proyecto
mkdir monitoring-stack && cd monitoring-stack
cd monitoring-stack 
mkdir prometheus
```
## Creacion de los yml
```bash
 nano docker-compose.yml

version: '3.8'

services:
  nextcloud:
    image: nextcloud
    container_name: nextcloud
    ports:
      - "8080:80"
    volumes:
      - nextcloud_data:/var/www/html
    depends_on:
      - db

  db:
    image: mariadb
    container_name: mariadb
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD= rootpass
      - MYSQL_PASSWORD= userpass
      - MYSQL_DATABASE= nextcloud
      - MYSQL_USER= nc_user
    volumes:
      - db_data:/var/lib/mysql

  prometheus:
    image: prom/prometheus
    container_name: prometheus
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana

  node-exporter:
    image: prom/node-exporter
    container_name: node-exporter
    ports:
      - "9100:9100"

  cadvisor:
    image: gcr.io/cadvisor/cadvisor
    container_name: cadvisor
    ports:
      - "8081:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro

volumes:
  nextcloud_data:
  db_data:
  grafana_data:

```

```bash
# cd prometheus
nano prometheus.yml
```

##Levantar los servicios

```bash
docker compose up -d
```



## Verificar interfaces web

- **Nextcloud:** http://localhost:8080

- **Grafana:** http://localhost:3000  
  Usuario: `admin`  
  Contraseña: `admin`

- **Prometheus:** http://localhost:9090

---

## Configurar Prometheus

En Prometheus, ve a `http://localhost:9090/targets` para confirmar que Node Exporter y cAdvisor aparecen como targets activos.

---

## Configurar Grafana

# Creacion del data source
1. Accede a `http://localhost:3000`
2. Ve a **Add new data source**
3. Selecciona **Prometheus**
4. En URL, escribe: `http://prometheus:9090`


![Imagen de Referencia (alt text)](/img/1.png)


#Creacion del dashboard


1. Ve a la seccion Dashboard
2. Dale en importar y en el id pondremos 1860
3. En el datasource pondremos el que acabamos de agregar.

![Imagen de Referencia (alt text)](/img/2.png)

4. resultado:

![Imagen de Referencia (alt text)](/img/3.png)

Si somos observadores, este dashboard solo es para si mismo y no para nextcloud. Por lo tanto haremos lo mismo pero con el id: 14282

5. resultado de todos los contenedores:

![Imagen de Referencia (alt text)](/img/4.png)


## Recursos adicionales

- **Guía paso a paso completa:** [Descargar instrucciones detalladas](https://estunanleonedu-my.sharepoint.com/:w:/g/personal/lramirez_loaisiga23_est_unanleon_edu_ni/EXhROJQFuDhIsUsAdh5_73IBgOiNWtvLt0pJhQadb4wSIA?e=rSm0Gh)  
- **Código fuente en GitHub:** [Ver repositorio](https://github.com/PavelR31/monitoreo-stack)
