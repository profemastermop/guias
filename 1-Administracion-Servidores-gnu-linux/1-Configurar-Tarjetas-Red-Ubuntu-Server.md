# Instructivo: Preparar Infraestructura de Trabajo

**Hecho Por:** MSc. Andrés Alberto Cortés Fuentes  
**Universidades:** UNA - UTN - UCR Sedes Liberia 

Encendemos la Máquina Virtual.

Nos conectamos de manera remota a través de **MobaXterm** por **SSH** al
servidor para proceder con las siguientes labores administrativas.

## Convertirse en superusuario

``` bash
sudo su
```

Ponemos la clave del usuario de la instalación del Sistema Operativo y
presionamos **ENTER**.

Pasamos de:

``` text
$
#
```

## Actualizar repositorios (OPCIONAL)

``` bash
apt update
```

## Actualizar paquetes instalados (OPCIONAL)

``` bash
apt upgrade
```

## Instalar net-tools (OPCIONAL)

``` bash
apt install net-tools
```

## Ver direcciones IP

``` bash
ifconfig
```

## Configurar zona horaria (OPCIONAL)

``` bash
date
timedatectl
timedatectl set-timezone America/Costa_Rica
timedatectl
date
```

## Configuración de Netplan

Editar el archivo:

``` bash
vim /etc/netplan/50-cloud-init.yaml
```

Borrar todo el contenido y pegar la siguiente configuración:

``` yaml
# This is the network config written by 'subiquity'
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: false
      addresses:
      - 192.168.100.1/24
      nameservers:
        addresses:
        - 192.168.100.1
        search: []
    enp0s9:
      dhcp4: false
      addresses: [192.168.56.190/24]
    enp0s10:
      dhcp4: true
      optional: true
      dhcp4-overrides:
        use-routes: false
        use-dns: false

```

Qué hace cada línea del bloque de la interfaz **enp0s10**

***dhcp4: true*** - define que el direccionamiento se va recibir de manera dinámica

***optional: true*** - el arranque no espera a que esa interfaz reciba IP; sin esto tarda dos minutos 

***use-routes: false*** - acepta la dirección pero ignora la ruta por defecto que anuncia el DHCP 

***use-dns: false*** - ignora el DNS que anuncia el DHCP, para no pisar el resolv.conf

## Ahora Proceder a Apagar la Máquina Virtual.

## Configuración de VirtualBox

Configurar tres adaptadores de red en el siguiente orden:

1.  **Adaptador 1:** NAT (salida a Internet).
2.  **Adaptador 2:** Red Interna.
3.  **Adaptador 3:** Solo Anfitrión.
    -   En **Avanzadas → Modo promiscuo**, seleccionar **Permitir
        Todo**.

Encender nuevamente la Máquina Virtual y verificar el direccionamiento
IP, así como la conectividad a Internet.

