# Instructivo: Instalación y Configuración de Servidor LAMP y Joomla

**Hecho Por:** MSc. Andrés Alberto Cortés Fuentes  
**Universidades:** UNA - UTN - UCR Sedes Liberia  

**Link de referencia:** [DigitalOcean - Instalación LAMP en Ubuntu 22.04](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-22-04)

---

## 🖥️ INSTALACIÓN SERVIDOR WEB - LAMP (Linux - Apache - MySQL - PHP)

```bash
date
timedatectl
timedatectl set-timezone "America/Costa_Rica"
timedatectl
date
apt update
```

### Instalación de Apache2
```bash
apt install apache2
```

### Verificamos que el servicio esté en ejecución
```bash
/etc/init.d/apache2 status

o

service apache2 status
```
### Ver la versión de Apache2
```bash
apache2 -v
```

Verificación del servicio desde el navegador del cliente:  
👉 `http://Ip_Servidor`

### Instalación de MySQL
```bash
apt install mysql-server php-mysql
```

### Ver la versión de MySQL
```bash
mysql --version
```
### Ingresar a la consola de administración del gestor de BD
```bash
mysql -u root -p

o

mysql

```

Para salir de la consola de mysql escribir y ejecutar:

```bash
exit;
```

### Instalación de PHP y extensiones
```bash
apt install php libapache2-mod-php

apt install php-zip php-gd php-intl php-pgsql
```

### Ver la versión de PHP
```bash
php -v
```

Crear página para ver datos del paquete vía web

```bash
vim /var/www/html/info.php
```

Contenido del archivo:
```php
<?php
phpinfo();
?>
```

Reinicio de servicio:
```bash
/etc/init.d/apache2 restart
```

Ver el estado del servicio:
```bash
/etc/init.d/apache2 status
```

Ver los parámetros de PHP desde el navegador del cliente:  
👉 `http://Ip_Servidor/info.php`

---

## 🌐 INSTALACIÓN GESTOR DE CONTENIDOS: JOOMLA

```bash
mkdir /datos

cd /datos

mkdir /sitios

cd /sitios

mkdir /joomla

cd /datos

wget https://github.com/joomla/joomla-cms/releases/download/5.2.0/Joomla_5.2.0-Stable-Full_Package.zip

ls -lt

mv Joomla_5.2.0-Stable-Full_Package.zip /datos/sitios/joomla

ls -lt

apt install zip unzip

cd /datos/sitios/joomla

ls -lt

unzip /datos/sitios/joomla/Joomla_5.2.0-Stable-Full_Package.zip -d /datos/sitios/joomla

ls -lt

cd /datos

ls -lt

chown www-data:www-data sitios/ -R

ls -lt

chmod 755 sitios/ -R

ln -s /datos/sitios/joomla/ /var/www/html/labweb

cd /var/www/html

ls -lt
```

El siguiente paquete es muy importante de instalar, ya que de lo contrario la interface de instalación de Joomla no nos va levantar en el navegador
```bash
apt install php-simplexml
```

Reiniciamos y validamos que el servicio esté en ejecución para continuar:

```bash
/etc/init.d/apache2 restart
/etc/init.d/apache2 status
```

---

## 🗄️ PREPARACIÓN DEL SERVIDOR MySQL PARA JOOMLA

Comando para el ingreso a la consola de administración.

Presionamos Enter para ingresar, ya que por defecto el ingreso para el usuario ROOT del Sistema Operativo está habilitado por medio de la directiva auth_socket

```bash
mysql -u root -p
```

Ver las bases de datos existentes

```bash
show databases;
```

Crear una nueva BD
```bash
CREATE DATABASE joomladb;
show databases;
```

Ver los usuarios existentes en el servidor de BD:
```bash
select User, Host, Plugin from mysql.user;
```

Creación de un nuevo usuario
```bash
CREATE USER joomlauser@localhost IDENTIFIED BY 'Linux20!';
```

Ver los usuarios existentes en el servidor de BD:
```bash
select User, Host, Plugin from mysql.user;
```

Asginar permisos al nuevo usuario
```bash
GRANT ALL PRIVILEGES ON joomladb.* TO 'joomlauser'@'localhost' WITH GRANT OPTION;
```

Ver los permisos asignados a los usuaios del servidor de BD
```bash
select u.User,Db from mysql.user u,mysql.db d where u.User=d.User;
```

Limpiar la cache de privilegios dentro del servidor de BD
```bash
FLUSH PRIVILEGES;
```

Salir de la consola de administración
```bash
quit
```

NOTA: Para cambiar la clave:
```sql
update user set authentication_string='Linux20!' where User='NombreUsuario';
flush privileges;
GRANT USAGE ON *.* TO NombreUsuario IDENTIFIED BY Linux20!;
```


Ahora nos vamos al navegador de la maquina fisica o anfitrion y escribimos la direccion ip del servidor mas /labweb por ejemplo: 192.168.56.X/labweb
 
👉 `http://192.168.56.X/labweb`

La IP del servidor la prodremos observar ejecutando el comando (Tener la interfaz de red identificada):
```bash
ip addr: 
```

Y continuamos el proceso desde el navegador...

### Parámetros de instalación Joomla:

Parte I
- Mi Primer Sitio

Parte II

- gnu  
- gnu
- If5100Log2M23!
- user@domain.com

Parte III
- Tipo de base de datos - opción por defecto
- Nombre hospedaje - opción por defecto
- joomlauser
- Linux20!
- joomladb
- Prefijo por defecto
- Conexión cifrada - opción por defecto


Ver los logs de la BD en caso de errores::
```bash
tail -f /var/log/mysql/error.log
```

Ver los logs de la BD de las ejecuciones:
```bash
cd /var/lib/mysql/
ls binlog.* 
```

Ahora ver el archivo binlog.0000X más reciente, este es el que termina con el número más alto:
Ejecutar:
```bash
mysqlbinlog --read-from-remote-server --stop-never binlog.00000X
```

---

## 🔒 HARDENING BÁSICO SERVIDOR WEB

### Borrar archivos por defecto
```bash
cd /var/www/html
rm -rf index.html info.php
```

Crear `index.html` vacío:
```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Blank</title>
</head>
<body></body>
</html>
```

### Securizar Joomla
```bash
cd /datos/sitios/joomla
rm -rf installation
```

### Securizar MySQL
```bash
mysql -u root -p
SELECT user, host, plugin from mysql.user;
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Mt9*Nr%3$hv!';
mysql_secure_installation
ALTER USER 'root'@'localhost' IDENTIFIED WITH auth_socket;
```

### Ocultar información de Apache2
Editar `/etc/apache2/conf-available/security.conf`:
```
ServerTokens Prod
TraceEnable off
ServerSignature Off
```

### Ocultar información de PHP
Editar `/etc/php/8.1/apache2/php.ini` según [Hosting Diario - Securizando PHP](https://hostingdiario.com/securizando-php/)

### Permisos de seguridad
```bash
find . -type d -exec chmod 755 '{}' \;
find . -type f -exec chmod 444 '{}' \;
```

### Cambiar palabras dentro de archivos
```bash
find . -name *.php | xargs perl -pi -e 's/palabra_a_reemplazar/palabra_a_asignar/g'
:%s/go.cr/onmicrosoft.com/g
```

---
**Fin del Instructivo**
