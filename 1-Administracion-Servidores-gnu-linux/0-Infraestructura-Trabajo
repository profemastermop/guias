# Instructivo: Preparar Infraestructura de Trabajo

**Hecho Por:** MSc. Andrés Alberto Cortés Fuentes  
**Universidades:** UNA - UTN - UCR Sedes Liberia 

> **RECOMENDACIÓN:** CREAR UN PUNTO DE RESTAURACIÓN EN SUS EQUIPOS
> WINDOWS, PARA PODER REALIZAR UN ROLLBACK!!!

## Descargas

-   VirtualBox: https://www.virtualbox.org/wiki/Downloads
-   En ocasiones es necesario instalar **VC_redist.x64** (buscar en
    Google).
-   Ubuntu Server:
    -   https://releases.ubuntu.com/24.04/
    -   https://releases.ubuntu.com/24.04/ubuntu-24.04.2-live-server-amd64.iso
-   Putty: https://www.putty.org/
-   MobaXterm: https://mobaxterm.mobatek.net/download-home-edition.html
-   WinSCP: https://winscp.net/eng/downloads.php

------------------------------------------------------------------------

# PARTE I

Nos conectamos de manera remota a través de SSH al servidor utilizando
la interfaz con direccionamiento **192.168.56.X**.

Indicar usuario y contraseña.

## Convertirse en superusuario

``` bash
sudo su
```

Introducir la contraseña del usuario de instalación.

Cambio de prompt:

``` text
$
#
```

## Actualizar el servidor

``` bash
apt update
apt upgrade
```

## Configurar zona horaria

``` bash
date
timedatectl
timedatectl set-timezone America/Costa_Rica
timedatectl
date
```

## Instalar herramientas de red

``` bash
apt install net-tools
```

## Ver direcciones IP

``` bash
ifconfig
```

------------------------------------------------------------------------

# PARTE II

## Configurar tarjeta de red

``` bash
sudo su
vim /etc/netplan/50-cloud-init.yaml
```

``` yaml
enp0s8:
  dhcp4: false
  addresses: [192.168.56.190/24]
```

Aplicar cambios:

``` bash
netplan apply
```

------------------------------------------------------------------------

# PARTE III

## Configuración del FQDN

Editar **/etc/hosts**:

``` text
127.0.0.1 localhost
192.168.56.190 server.gnulinux.mop server
```

Editar **/etc/hostname**:

``` text
server.gnulinux.mop
```

Reiniciar:

``` bash
init 6
```

------------------------------------------------------------------------

# PARTE IV

## 1. Copiar archivos desde Windows mediante SCP

Desde el CMD de Windows:

``` bash
scp -rv Nombre-Carpeta gnu@IP_SERVIDOR:/tmp
```

En el servidor:

``` bash
cd /tmp
ls -lt
```

Mover carpeta a `/datos`:

``` bash
chown gnu /datos -R
mv /tmp/Nombre-Carpeta /datos/
cd /datos/
ls -lt
```

## 2. Transferencia con WinSCP

Copiar el directorio **Nombre-Carpeta** desde Windows al directorio
**/datos** del servidor.

------------------------------------------------------------------------

# FIN

