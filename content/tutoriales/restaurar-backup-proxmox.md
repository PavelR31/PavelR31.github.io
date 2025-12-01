---
title: "Restaurar Backup Proxmox"
date: 2025-11-30T19:26:34-06:00
draft: true
tags: ["proxmox", "linux", "backup", "firewall"]
categories: ["SysAdmin"]
---

Recientemente necesité mover unas máquinas virtuales  desde un backup guardado en mi PC local hacia una instalación limpia de Proxmox VE. Aquí explico cómo hacerlo.

## Escenario
* **Origen:** Tengo el archivo `.vma.zst` en mi computadora local (Linux/Mac/WSL).
* **Destino:** Un servidor Proxmox nuevo (IP: `192.168.1.176`).
* **Objetivo:** Restaurar la VM para que funcione en el nuevo servidor.

## 1. Subir el archivo al servidor
Proxmox espera encontrar los backups en una ruta específica. Por defecto para el almacenamiento `local` es `/var/lib/vz/dump/`.

Usamos `rsync` para subirlo. En este caso, mis archivos se llaman `debian_dns.vma.zst - firewall_pfsense.vma.zst - ldap_ubuntu.vma.zst  -  nas_truenas.vma.zst`.

```
# Desde la PC origen hasta el servidor proxmox. (ambas en el mismo segmento de red privada).
rsync -avP  *.vma.zst root@192.168.1.176:/var/lib/vz/dump/
```

## 2. Verificar el archivo subido en Proxmox 
Nos podemos conectar por `ssh` al servidor.

```
ssh root@192.168.1.176
cd /var/lib/vz/dump/
ls -lh 
``` 

![Imagen de Referencia (alt text)](/img/tutorialTerminal.png)


## 3. Restaurar desde la interfaz web

### 3.1 Acceso a Proxmox VE y ubicación del módulo de respaldos

Para iniciar el proceso de restauración, primero ingresamos a la interfaz web de **Proxmox VE** utilizando nuestras credenciales de administrador.

Una vez dentro del panel principal, en el árbol de navegación lateral seguimos la ruta:

**`pve` → `local (pve)` → `Backups`**

En esta sección se muestran todos los respaldos almacenados localmente en el nodo, tanto de máquinas virtuales como de contenedores. Desde aquí podremos seleccionar el archivo de backup correspondiente y continuar con el proceso de restauración.


![Imagen de Referencia (alt text)](/img/backups.png)

### 3.2 Selección del respaldo y ejecución de la restauración

Una vez dentro del apartado **Backups**, localizamos en la lista el archivo correspondiente a la **máquina virtual que deseamos restaurar**.  
Podemos identificarla por su nombre, ID de VM, fecha del respaldo o tipo de archivo (`.vma.zst`).

Cuando tengamos el respaldo correcto seleccionado, hacemos clic en el botón **Restore** ubicado en la parte superior de la interfaz.

Este botón abrirá la ventana de restauración, desde donde podremos definir el nodo destino, el almacenamiento donde se desplegará la VM y otros parámetros antes de iniciar el proceso.


![Imagen de Referencia (alt text)](/img/restaurarProxmox.png)

### 3.3 Finalización del proceso y confirmación

Una vez configurados los parámetros de restauración y confirmado el proceso, Proxmox iniciará la tarea de forma automática.  
Podemos ver el progreso en tiempo real desde la pestaña **Tasks** o en la ventana emergente que muestra los logs de la operación.

Cuando la restauración haya finalizado, la máquina virtual aparecerá nuevamente en el árbol lateral con su **ID asignado**, lista para encenderla y validar que todo funcione correctamente.

Con esto el proceso queda completado.  
A partir de este punto, la VM ya está totalmente operativa en el nuevo servidor Proxmox y se puede comenzar a trabajar con ella sin necesidad de pasos adicionales.




![Imagen de Referencia (alt text)](/img/proceso.png)


> **Nota importante:**  
> En este tutorial utilizamos nombres simplificados como `debian_dns.vma.zst` o `nas_truenas.vma.zst` únicamente para facilitar la lectura del ejemplo.  
> Sin embargo, los respaldos generados por **Proxmox VE** siempre tendrán una estructura de nombre específica que incluye el ID de la máquina virtual, el tipo de backup y la fecha.  
>  
> Un nombre real generado automáticamente por Proxmox luce así:
> ```
> vzdump-qemu-102-2025_11_30-18_43_44.vma.zst
> ```
> Al momento de subir o restaurar el archivo, es indispensable **mantener el nombre original**, ya que Proxmox utiliza esta estructura para identificar correctamente la VM, su configuración y los metadatos del respaldo.


## Conclusión

Restaurar máquinas virtuales en Proxmox VE a partir de un archivo de respaldo (`.vma.zst`) es un proceso directo y altamente confiable, ya que el formato nativo de Proxmox conserva no solo los discos virtuales, sino también la configuración completa de la VM (CPU, RAM, red, hardware virtual, BIOS/UEFI, etc.). Esto lo convierte en el método ideal cuando se requiere migrar instalaciones completas, recuperar un entorno de trabajo tras un fallo o mover laboratorios entre servidores.

A diferencia de otros formatos de exportación —como **OVF/OVA**, imágenes **QCOW2**, **RAW** o discos en formatos genéricos—, los backups de Proxmox no requieren pasos adicionales de importación, conversión ni recreación manual de la configuración. Con un archivo `.vma.zst` basta con subirlo a la carpeta correspondiente y restaurarlo desde la interfaz web para tener la máquina lista y funcional en pocos minutos.

Este tipo de restauración sobresale por su sencillez, seguridad y coherencia, ya que conserva el estado exacto de la VM al momento del respaldo. Una vez completado el proceso, la máquina virtual queda totalmente operativa y lista para continuar su funcionamiento sin necesidad de configuraciones extra.
