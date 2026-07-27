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

```bash
cp rules /etc/shorewall/
vim /etc/shorewall/rules
```
(No se edita por ahora).

---

### Archivo `stoppedrules`

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

```bash
cp zones /etc/shorewall/
vim /etc/shorewall/zones
```
(No requiere cambios).

---

### Archivo `shorewall.conf`

```bash
vim /etc/shorewall/shorewall.conf
```

Modificar las siguientes líneas:

```bash
LOG_MARTIANS=NO
IP_FORWARDING=Yes
```

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

- **IP:** 192.168.100.10  
- **Máscara:** 255.255.255.0  
- **Gateway:** 192.168.100.3  
- **DNS:** 192.168.100.1, 8.8.8.8, 8.8.4.4

---

## 5. Apertura de Puertos

Editar el archivo `/etc/shorewall/rules`.

### a. SSH desde máquina física
```bash
SSH(ACCEPT)     dmz             $FW
```

### b. DNS para Red Interna
```bash
DNS(ACCEPT)     loc             $FW
```

### c. HTTP (Web) para Red Interna
```bash
#
#       Accept HTTP connections
#
HTTP(ACCEPT)     loc             $FW
```

---

Con esto se permite acceso a **Apache2** y salida a Internet desde la zona **loc**.