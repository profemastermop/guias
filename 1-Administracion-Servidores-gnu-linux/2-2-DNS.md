# Instructivo: Curso Administración de Servidores con Software Libre

**Hecho Por:** MSc. Andrés Alberto Cortés Fuentes  
**Universidades:** UNA - UTN - UCR Sedes Liberia 

# Guía de Configuración de Servicio DNS IPv4

## 1. Instalación y Preparación
Instalamos el paquete del dns (bind9) con la instrucción:
```bash
apt install bind9
```
Nos movemos al directorio de trabajo (el cual es creado con la instalación del paquete bind9):
```bash
cd /etc/bind
```

## 2. Configuración del Archivo Principal
Primeramente, editamos el archivo:
```bash
vim /etc/bind/named.conf
```
Y comentamos la siguiente línea (agregando un `#` al inicio):
```text
#include "/etc/bind/named.conf.default-zones";
```
Dentro de este directorio, los archivos que inician con `db` son archivos de ejemplo de las zonas:
* `db.#`: zona de búsqueda inversa.
* `db.xxxx`: zona de búsqueda directa.

## 3. Definición de Zonas
Las zonas se definen en el archivo `named.conf.local` agregando la definición tanto de la zona directa como la inversa.
```bash
vim named.conf.local
```
Agregue al final del archivo la siguiente configuración:
```conf
zone "gnulinux.mop" {
        type master;
        notify yes;
        file "/etc/bind/gnulinux.mop.hosts";
};

zone "100.168.192.in-addr.arpa" {
        type master;
        notify yes;
        file "/etc/bind/192.168.100.rev";
};
```

## 4. Creación de los Archivos de Zona
Una vez definidas las dos zonas, creamos los archivos en los lugares respectivos haciendo una copia de los archivos de ejemplo para evitar digitar todo desde cero:
```bash
cp /etc/bind/db.local /etc/bind/gnulinux.mop.hosts
cp /etc/bind/db.127 /etc/bind/192.168.100.rev
```

## 5. Configuración de Zonas
### Zona de Búsqueda Directa
Editamos el archivo de la zona directa:
```bash
vim gnulinux.mop.hosts
```
Reemplazamos `localhost` por el nombre del dominio (`gnulinux.mop`). Puede utilizar el siguiente comando en Vim para reemplazar de forma masiva una parte del texto:
```vim
:%s/localhost/gnulinux.mop/g
```
Además, agregamos los registros tipo A que definen el nombre del host. Añada al final del archivo:
```text
principal       IN      A       192.168.100.1
cliente         IN      A       192.168.100.10
```

### Zona de Búsqueda Inversa
Editamos el archivo de la zona inversa:
```bash
vim 192.168.100.rev
```
Modificamos el `localhost` por el nombre de nuestro dominio (con el mismo comando de Vim utilizado anteriormente).
Agregamos los registros PTR para que el dns pueda resolver por medio de la dirección IP. Añada al final del archivo (recuerde el punto al final del dominio):
```text
1       IN      PTR     principal.gnulinux.mop.
10      IN      PTR     cliente.gnulinux.mop.
```

## 6. Validación de Sintaxis
Para probar que lo escrito en `named.conf.local` está correcto a nivel de estructura (si todo está bien, solo obtendremos un salto de línea):
```bash
named-checkconf named.conf.local
```
Verificamos la estructura de los archivos de zona:
```bash
named-checkzone gnulinux.mop /etc/bind/gnulinux.mop.hosts
named-checkzone in-addr.arpa /etc/bind/192.168.100.rev
```

## 7. Reinicio de Servicio y Monitoreo
Al realizar cambios, debemos reiniciar el servicio del dns:
```bash
service bind9 restart
```
*(Opcionalmente se puede detener con `stop` o iniciar con `start`)*.

En caso de errores o si el servicio no está activo (`service bind9 status`), revise el log del sistema:
```bash
tail -f /var/log/syslog
```

## 8. Configuración de Resolución (Ubuntu 13.04 en adelante)
Verificamos los servicios de resolución de nombres con:
```bash
resolvectl
```
En versiones recientes, `/etc/resolv.conf` es un enlace simbólico. Para configurar un DNS estático (tanto en servidor como en cliente):
```bash
mv /etc/resolv.conf /etc/resolv.conf_ORI
cp /etc/resolv.conf_ORI /etc/resolv.conf
vim /etc/resolv.conf
```
Y agregamos la siguiente línea al principio:
```text
nameserver 192.168.100.1
```

## 9. Prueba de Funcionamiento
Procedemos a probar el funcionamiento del DNS con el comando `nslookup` desde el cliente:
```bash
nslookup
> 192.168.100.1
> 192.168.100.10
> principal.gnulinux.mop
> cliente.gnulinux.mop
> www.google.com
> www.algoquenoexiste.com