# IF5100 Administración de Bases de Datos

**Profesor:** MSc. Andrés Alberto Cortés Fuentes  
**Universidades:** UNA - UTN - UCR Sedes Liberia  

---

# Implementación y Puesta en Producción: RAID5

------------------------------------------------------------------------

## Creación de la Máquina Virtual

Crear una máquina virtual con las siguientes características de discos
duros.

**NOTA:**\
Recordar marcar las opciones de: - Unidad de Estado Sólido (SSD) -
Conectable en Caliente

Asignar una tarjeta de red: - Solo Anfitrión

### Configuración de discos

-   **1 disco de 10G** para el Sistema Operativo
    -   1G → swap
    -   4G → /
    -   resto → /home
-   **3 discos de 20G** para uso del RAID-5

------------------------------------------------------------------------

## Configuración del Arreglo RAID

El nombre del arreglo debe ser:

**RAID5**

Con el siguiente particionamiento:

-   20G → /produccion
-   resto → /desarrollo

------------------------------------------------------------------------

## Monitoreo del Arreglo RAID

Una vez finalizada la instalación, nos conectamos por medio de la
terminal y ejecutamos el siguiente comando:

``` bash
while true; do mdadm -D /dev/md/RAID5; sleep 3; clear; done;
```

### Estructura del comando:

``` bash
mdadm -D /dev/md/NombreArreglo
```

------------------------------------------------------------------------

## Monitoreo de Logs del Sistema

Abrimos otra consola con MobaXterm en modo Split horizontal y
ejecutamos:

``` bash
tail -f /var/log/syslog
```

Esto permite observar en tiempo real lo que sucede en el servidor.

------------------------------------------------------------------------

## Simulación de Falla de Disco

Desde las propiedades de la máquina virtual:

1.  Eliminar la conexión del último disco del arreglo
2.  Observar el comportamiento en las terminales
3.  Analizar el estado del RAID

------------------------------------------------------------------------

## Reconstrucción del Arreglo RAID

Para reconstruir el arreglo:

1.  Volver a conectar el disco duro desde las propiedades de la máquina
    virtual
2.  Ejecutar el siguiente comando desde una terminal (Putty):

``` bash
mdadm /dev/md/RAID5 --add /dev/sdd
```

------------------------------------------------------------------------

## Análisis del Proceso

Observar desde la terminal y la consola de VirtualBox:

-   Estado del arreglo RAID
-   Proceso de reconstrucción
-   Logs del sistema

------------------------------------------------------------------------

## Link de referencia

https://www.digitalocean.com/community/tutorials/how-to-manage-raid-arrays-with-mdadm-on-ubuntu-22-04