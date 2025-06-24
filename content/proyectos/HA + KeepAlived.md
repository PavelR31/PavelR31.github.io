---
title: "Cluster Web Apache en Alta Disponibilidad con HAProxy + Keepalived (IP Flotante)"
date: 2025-06-23
tags: ["debian", "haproxy", "keepalived", ]
description: "Implementaremos un **cluster web Apache** con balanceo de carga usando HAProxy y redundancia mediante Keepalived para una IP virtual flotante. Ideal para entornos críticos donde la disponibilidad es prioridad."
---

## 🌐 Topología
1. **Cliente**: Usuario que accede al sitio web.
2. **IP Virtual (VIP)**: `172.22.0.100` (gestionada por Keepalived).
3. **Nodos HAProxy**:
   - `haproxy1` (MASTER): `172.22.0.5`
   - `haproxy2` (BACKUP): `172.22.0.4`
4. **Servidores Apache**:
   - `apache1`: `172.22.0.10`
   - `apache2`: `172.22.0.11`


![Imagen de Referencia (alt text)](/img/ha.png)

## 🛠️ Configuración Paso a Paso

### 1. Creación del Escenario en VirtualBox
- **SO**: Debian en 4 máquinas virtuales.
- **Red**: NAT Network en VirtualBox (Tools > Network > NAT Network > Create).

![Imagen de Referencia (alt text)](/img/5.jpeg)

### 2. Configuración de IPs Estáticas
Editar `/etc/network/interfaces` en cada VM:

```bash
auto enp0s3
iface enp0s3 inet static
address 172.22.0.5
netmask 255.255.255.0
gateway 172.22.0.1

```
### 3. Instalacion de los servivios
- **En nodos apache**
```bash
sudo apt update && sudo apt install apache2 -y
```
- **En nodos HAProxy (Master Y backup)**

```bash
sudo apt install haproxy keepalived -y
```

### 4. Configuracion de HAProxy
- **Editar /etc/haproxy/haproxy.cfg (en ambos HAProxy):**
```bash
frontend http_front
   bind *:80
   default_backend http_back

backend http_back
   balance roundrobin
   option httpchk
   server apache1 172.22.0.10:80 check
   server apache2 172.22.0.11:80 check
```
- **Editar el archivo  /etc/default/haproxy**
```bash
#Agregamos
ENBALED=1
```

- **Reiniciamos con**
```bash
systemctl restart haproxy
```

Realizamos exactamente lo mismo en el backup.


### 5. Configuración de keepalived
- **En la maquina master editamos el archivo  /etc/keepalived/keepalived.conf**

```bash
vrrp_instance VI_1 {
    state MASTER
    interface enp0s3
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        172.22.0.100
    }
}
```
- **En maquina backup**

```bash
vrrp_instance VI_1 {
    state BACKUP
    interface enp0s3
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        172.22.0.100
    }
}
```

### 6 🧾 Configuración de Apache

- En **cada servidor Apache**
 (`apache1` y `apache2`), realizamos los siguientes pasos:

📁 1. Subir Archivos del Proyecto

Copia tus archivos del sitio web (HTML, CSS, JS, etc.) al directorio raíz de Apache:

En mi caso un proyecto de react, por lo tanto pegue el proyecto. En tu caso, pones la ruta del proyecto.
```bash
sudo rm -rf /var/www/html/*
sudo cp -r /ruta/proyecto* /var/www/html/
```
- **Asegúrate de que los archivos tengan los permisos correctos:**
```bash
sudo chown -R www-data:www-data /var/www/html
```

- De ser necesario puedes crear .htaccess, en mi caso si lo es por usar react.
```bash
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteRule ^(.*)$ index.html [L]
</IfModule>

```

- Habilitamos la pagina y reinciamos el servicio

```bash
sudo a2enmod rewrite
sudo systemctl restart apache2
```

- Ejemplos de como se ve el server web.


![Imagen de Referencia (alt text)](/img/5.jpeg)
## Recursos adicionales

- **Guía paso a paso completa:** [Descargar instrucciones detalladas](https://estunanleonedu-my.sharepoint.com/:w:/g/personal/lramirez_loaisiga23_est_unanleon_edu_ni/Efk6zkyn1sdAshY1BEf0tOIBAhGBn_xQpv7lEv9zwGCUgA?e=hg0Ij0)  
