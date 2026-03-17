# IF5100 Administración de Bases de Datos

**Profesor:** MSc. Andrés Alberto Cortés Fuentes  
**Universidades:** UNA - UTN - UCR Sedes Liberia  

---

## ⚠️ RECOMENDACIÓN
Crear un **punto de restauración en Windows** antes de iniciar el laboratorio, para poder realizar un rollback si fuese necesario.

---

## 📥 Descargas Necesarias

- VirtualBox  
  https://www.virtualbox.org/wiki/Downloads  

- VC_redist.x64 (buscar en Google si es necesario)

- ISO Ubuntu Server 24.04  
  https://releases.ubuntu.com/24.04/  
  https://releases.ubuntu.com/24.04/ubuntu-24.04.2-live-server-amd64.iso  

- PgAdmin 4  
  https://www.pgadmin.org/download/pgadmin-4-windows/  

- Putty  
  https://www.putty.org/  

- MobaXterm  
  https://mobaxterm.mobatek.net/download-home-edition.html  

- WinSCP  
  https://winscp.net/eng/downloads.php  

---

# Instalación PostgreSQL en Ubuntu Server

Referencia:  
https://www.digitalocean.com/community/tutorials/how-to-install-postgresql-on-ubuntu-22-04-quickstart

Nos conectamos de manera remota a traves del protocolo SSH al servidor para proceder con las siguientes labores administrativas, para ello utilizamos la interface que tiene el direccionamiento 192.168.56.X

Una vez conectados nos convertimos en Super Usuario

```bash
sudo su
```
Ponemos la clave del usuario de la instalación del Sistema Operativo y presionamos la tecla ENTER

Actualizar servidor:

```bash
apt update
apt upgrade
```

Configurar zona horaria:

```bash
date 
timedatectl
timedatectl set-timezone America/Costa_Rica
timedatectl
date
```

Instalar herramientas de red:

```bash
apt install net-tools
ifconfig
```

Instalar PostgreSQL:

```bash
apt install postgresql postgresql-contrib
```

Para ingresar a la consola de psql desde el usuario de la instalación de Ubuntu o ROOT lo podemos hacer de las siguientes dos maneras::

```bash
sudo -i -u postgres
psql
```
o bien:

```bash
sudo -u postgres psql
```

Cambiar clave:

```sql
alter user postgres with password 'linux';
```

---

# Habilitar Acceso Remoto

Editar:

```bash
vim /etc/postgresql/16/main/postgresql.conf
```

Modificar:

```
listen_addresses = '*'
```

Editar:

```bash
vim /etc/postgresql/16/main/pg_hba.conf
```

Agregar: Nota: la línea comentada la vamos a utilizar más adelante cuando se cree un docker de PgAdmin4

```
host    all    all    192.168.56.0/24    md5
#host    all    all    172.17.0.0/24      md5
```

Reiniciar servicio:

```bash
/etc/init.d/postgresql stop
/etc/init.d/postgresql status
/etc/init.d/postgresql start
/etc/init.d/postgresql status
```

Para saber la version de postgres que estamos utilizando:

```bash
gnu@svrbd:~$ sudo -i -u postgres
postgres@svrbd:~$ psql --version

O bien utilizando el super usuario:

root@svrbd:/home/gnu# psql --version
```
---

# PgAdmin 4 en Windows

Abrimos la aplicación de PgAdmin4 y realizamos una conexión desde la maquina física al servidor utilizando la dirección de red que empieza con 192.168.56.X


Crear Rol desde la terminal psql:

```sql
CREATE ROLE nombre-alumno WITH SUPERUSER CREATEDB CREATEROLE LOGIN ENCRYPTED PASSWORD 'linux';
```

Verificar:

```sql
\du
```

---

# Fin parte 1...

## Ejemplos comandos SQL para crear bases de datos: 

Descargar el directorio titulado: Ejercicio-Pedidos

Preparamos los permisos del directorio /datos en el servidor

```bash
cd /
chmod 777 datos/ -R
```

Dos alternativas para enviar el directorio al Servidor con Ubuntu Server

1. En un cmd de la máquina con Windows:

```bash
dir
cd Downloads
dir
```

Enviar archivos de Windows a Gnu/Linux

```bash
scp -rv Ejercicio-Pedidos gnu@IP_SERVIDOR:/tmp
```

Desde el servidor estando en la consola del MobaXterm 

```bash
cd /tmp
ls -lt
```

Y vamos a ver la carpeta que acabamos de enviar desde la máquina con Windows.

Desde la consola del MobaXterm mover la carpeta Ejercicio-Pedidos a /datos

```bash
mv /tmp/Ejercicio-Pedidos /datos/
cd /datos/
ls -lt
```

Asignamos los permisos correspondientes:

```bash
chmod 777 Ejercicio-Pedidos -R
```

2. Con WinSCP pasamos el directorio Ejercicio-Pedidos de Windows al directorio /datos del Servidor


Para ejecutar los scripts ubicados en /datos/Ejercicio-Pedidos

Desde el usuario gnu de la instalacion:

```bash
gnu@svrbd:~$ sudo -i -u postgres
postgres@svrbd:~$ cd /datos/Ejercicio-Pedidos
postgres@svrbd:/datos/Ejercicio-Pedidos$ psql
postgres=# \i createpedidos.sql
postgres=# \i insertpedidos.sql
```

NOTA: Ver referencia tipos de Datos en psql https://www.postgresql.org/docs/14/datatype.html


# Fin Parte 2
