---
title: "SecScope - API 🛡️"
date: 2025-07-03
tags: ["linux", "scripting", "python"]
description: "Api para monitoreo de servidores linux."
---

API desarrollada con **FastAPI** para realizar un escaneo de seguridad del sistema Linux, incluyendo:

- Información del sistema (RAM, CPU, uptime, distro)
- Servicios en ejecución
- Binarios con permisos especiales (SUID/SGID)
- Auditoría del servicio SSH

## 📦 Requisitos

- Python 3.9+
- `venv` activado
- Linux con acceso a información del sistema

## 📁 Estructura del proyecto

```bash

secscope/
├── api.py
├── utils/
│ ├── sysinfo.py
│ ├── services.py
│ ├── permissions.py
│ └── ssh_audit.py
└── venv/
```

## Instalación y ejecución

```bash
# 1. Crear entorno virtual si no existe
python3 -m venv venv

# 2. Activar entorno
source venv/bin/activate

# 3. Instalar dependencias
pip install -r requirements.txt
# o si no tienes requirements.txt:
pip install fastapi uvicorn psutil

# 4. Ejecutar la API
uvicorn api:app --reload

```
- **Repositorio** [Descarga el repositorio para probar la API](https://github.com/PavelR31/SecScope)



Una vez iniciada, accede a la documentación automática en:

Swagger UI: http://localhost:8000/docs

Redoc: http://localhost:8000/redoc



## Configuracion de Grafana

Abre Grafana y ve a "Import dashboard"

Pega el contenido del archivo JSON o cárgalo directamente

Asocia el panel a tu fuente de datos


- **Json Grafana** [Descargar instrucciones detalladas](https://estunanleonedu-my.sharepoint.com/:u:/g/personal/lramirez_loaisiga23_est_unanleon_edu_ni/ES2qZ9kwBfdGpOt1BFED6WsB4N4jTRCZchrYZTTu0MPTrA?e=BVcPJZ) ## Descarga el repositorio 

## ✅ ¿Qué deberías ver en Grafana?
 - **El dashboard de Grafana mostrará paneles como:**

- **Uso de CPU y RAM en tiempo real**

- **Servicios en ejecución**

- **Auditoría de configuración SSH**

- **Detalles del sistema (hostname, arquitectura, etc.)**



# Vista previa del dashboard

![Ejemplo del panel de control con métricas del sistema](/img/SecScope.jpeg)

---

# Contribuciones

¡Bienvenidas las contribuciones!

- Haz fork del proyecto  
- Crea una rama: `git checkout -b feature/nueva-funcionalidad`  
- Haz commit: `git commit -am 'Añade nueva funcionalidad'`  
- Haz push: `git push origin feature/nueva-funcionalidad`  
- Abre un Pull Request  
