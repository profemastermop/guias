# Instructivo Configuración de Shorewall

**Hecho Por:** MSc. Andrés Alberto Cortés Fuentes  
**Universidades:** UNA - UTN - UCR Sedes Liberia  

---
## Recursos Conceptuales

### Qué es un Firewall
https://www.youtube.com/watch?v=-dRabbTm9C4

### Qué es una DMZ
https://youtu.be/dqlzQXo1wqo


---

## 1. Instalación del Servicio

Shorewall es una herramienta de seguridad informática para Linux que permite configurar y administrar un firewall (cortafuegos). Utiliza el sistema de filtrado de paquetes del kernel de Linux para controlar qué conexiones de red se permiten y cuáles se bloquean.

```bash
apt install shorewall
cd /etc/shorewall
ls -lt
cd /usr/share/doc/shorewall/examples
ls -lt
cd three-interfaces
ls -lt
```

---

## 2. Copiar y Modificar Archivos de Configuración

### Archivo `interfaces`

En Shorewall, el archivo interfaces sirve para indicarle al firewall qué interfaces de red tiene el equipo y a qué zona de seguridad pertenece cada una.

```bash
cp interfaces /etc/shorewall/
vim /etc/shorewall/interfaces
```

**Importante:**  
Debemos tener claras las siguientes asociaciones:

- `net`: interfaz configurada como **NAT** (VirtualBox).  
- `loc`: interfaz configurada como **Red Interna** (VirtualBox).  
- `dmz`: interfaz configurada como **Solo Anfitrión** (VirtualBox).  

**Ejemplo de archivo:**

```bash
net     enp0s3          tcpflags,dhcp,nosmurfs,routefilter,sourceroute=0
loc     enp0s8          tcpflags,nosmurfs,routefilter
dmz     enp0s9          tcpflags,nosmurfs,routefilter
```

---

### Archivo `snat`

En Shorewall, el archivo snat sirve para configurar el NAT de origen (Source Network Address Translation). Permite cambiar la dirección IP de origen de los paquetes que salen de una red hacia otra.

```bash
cp snat /etc/shorewall/
vim /etc/shorewall/snat
```

**Ejemplo de archivo:**

```bash
MASQUERADE              10.0.0.0/8,\
                        169.254.0.0/16,\
                        172.16.0.0/12,\
                        192.168.0.0/16          enp0s3
```

---

### Archivo `policy`

En Shorewall, el archivo policy sirve para definir las políticas predeterminadas de tráfico entre las zonas de red. Es decir, establece qué debe hacer el firewall cuando una conexión no coincide con ninguna regla específica del archivo rules.

```bash
cp policy /etc/shorewall/
vim /etc/shorewall/policy
:se nu
```

Borramos desde la línea 16 y pegamos el siguiente bloque:

```bash
#REGLAS PARA ADAPTADOR RED INTERNA
loc             net             ACCEPT
loc             $FW             REJECT          $LOG_LEVEL
loc             dmz             REJECT          $LOG_LEVEL
loc             all             DROP            $LOG_LEVEL

#REGLAS PARA LA ZONA DEL FIREWALL
$FW             net             ACCEPT
$FW             loc             REJECT          $LOG_LEVEL
$FW             dmz             REJECT          $LOG_LEVEL
$FW             all             DROP            $LOG_LEVEL

#REGLAS PARA EL ADAPTADOR RED SOLO ANFITRION
dmz             net             ACCEPT
dmz             loc             REJECT          $LOG_LEVEL
dmz             $FW             REJECT          $LOG_LEVEL
dmz             all             DROP            $LOG_LEVEL

#REGLAS PARA EL ADAPTADOR NAT
net             loc             REJECT          $LOG_LEVEL
net             $FW             REJECT          $LOG_LEVEL
net             dmz             REJECT          $LOG_LEVEL
net             all             DROP            $LOG_LEVEL

# THE FOLLOWING POLICY MUST BE LAST
all             all             REJECT          $LOG_LEVEL
```

---

### Archivo `rules`

En Shorewall, el archivo rules sirve para definir las reglas específicas que permiten, rechazan o bloquean determinados tipos de tráfico de red. Por ejemplo, permitir el acceso a una página web, habilitar SSH para administrar un servidor o bloquear conexiones a determinados puertos.

```bash
cp rules /etc/shorewall/
vim /etc/shorewall/rules
```
(No se edita por ahora).

---

### Archivo `stoppedrules`

En Shorewall, el archivo stoppedrules sirve para definir reglas de firewall que se aplican cuando Shorewall está detenido (stopped), según la versión y la configuración utilizada.

```bash
cp stoppedrules /etc/shorewall/
vim /etc/shorewall/stoppedrules
```

**Ejemplo de archivo:**

```bash
#ACTION         SOURCE          DEST            PROTO   DEST            SOURCE
#                                                       PORT(S)         PORT(S)
ACCEPT          enp0s8          -
ACCEPT          -               enp0s8
ACCEPT          enp0s9          -
ACCEPT          -               enp0s9
```

---

### Archivo `zones`

En Shorewall, el archivo zones sirve para definir las zonas de seguridad de la red. Estas zonas permiten agrupar equipos o interfaces según el nivel de confianza o la función que cumplen, para que el firewall pueda controlar el tráfico entre ellas.

```bash
cp zones /etc/shorewall/
vim /etc/shorewall/zones
```
(No requiere cambios).

---

### Archivo `shorewall.conf`

El archivo shorewall.conf sirve para configurar los parámetros generales de Shorewall, el firewall de Linux. Es decir, controla cómo funciona Shorewall y qué características utiliza para proteger la red.

```bash
vim /etc/shorewall/shorewall.conf
```

Modificar las siguientes líneas:

```bash
LOG_MARTIANS=NO
IP_FORWARDING=Yes
```

En Shorewall, estos dos parámetros controlan aspectos clave sobre cómo el sistema maneja el tráfico de red y los registros de seguridad:

LOG_MARTIANS=NO
Los "martians" (marcianos) son paquetes de red que provienen de direcciones IP que técnicamente no deberían existir o no deberían verse en esa interfaz (por ejemplo, paquetes con direcciones IP de origen privadas o imposibles según la tabla de enrutamiento, como tráfico de internet que llega con una IP local). 

El valor NO: Indica que Shorewall no registrará (no guardará en los logs del sistema) estos paquetes extraños o inválidos cuando el núcleo de Linux los descarte automáticamente.

IP_FORWARDING=Yes
Controla si el sistema Linux tiene permitido reenviar tráfico (actuar como un router o pasarela) entre diferentes interfaces de red.

El valor Yes: Le indica a Shorewall que debe activar el reenvío de paquetes a nivel del núcleo (net.ipv4.ip_forward = 1). Esto es indispensable si el equipo con Shorewall funciona como un enrutador, pasarela (gateway) o comparte internet con otras subredes o contenedores (por ejemplo, si conecta tu red local LAN hacia la WAN/Internet). Si estuviera en No, el servidor bloquearía cualquier intento de pasar tráfico de una red a otra.

---

### Activar el Firewall

```bash
vim /etc/default/shorewall
```
Cambiar la línea:

```bash
startup=1
```

---

## 3. Reinicio y Verificación

```bash
init 6
/etc/init.d/shorewall status
/etc/init.d/shorewall start
tail -f /var/log/syslog
```

---

## 4. Configuración del Cliente

- **IP:** Varlor entre 1 y 254 del segmento de red utilizado
- **Máscara:** 255.255.255.0  
- **Gateway:** Ip Interface Red Interna del Servidor  
- **DNS:** Ip Interface Red Interna del Servidor, 8.8.8.8, 8.8.4.4

**NOTA:** El valor del parámetro option routers del archivo de configuración de DHCP debe ser la Ip de la Interface Red Interna del Servidor

---

## 5. Apertura de Puertos

Editar el archivo `/etc/shorewall/rules`.

### a. SSH desde máquina física (Puerto 22)
```bash
SSH(ACCEPT)     dmz             $FW
```

### b. DNS para Red Interna (Puerto 53)
```bash
DNS(ACCEPT)     loc             $FW
```

### c. HTTP (Web) para Red Interna (Puerto 80)
```bash
#
#       Accept HTTP connections
#
HTTP(ACCEPT)     loc             $FW
```
### d. DHCP para Red Interna (Puertos 67 y 68)
```bash
DHCPfwd(ACCEPT)     loc             $FW
DHCPfwd(ACCEPT)     $FW             loc
```

---

Con esto se permite acceso a **Apache2**, entrega de direccionamiento dinámico y salida a Internet desde la zona **loc**.