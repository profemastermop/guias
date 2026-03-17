# IF5100 Administración de Bases de Datos

**Profesor:** MSc. Andrés Alberto Cortés Fuentes  
**Universidades:** UNA - UTN - UCR Sedes Liberia 

---

# Implementación y Puesta en Producción: Docker de PostgreSQL

------------------------------------------------------------------------

## 1. Crear una máquina virtual nueva

Creamos una máquina virtual nueva.

------------------------------------------------------------------------

## 2. Actualizar la zona horaria

Actualizamos la zona horaria para establecer la fecha y hora correcta.

``` bash
date
timedatectl
timedatectl set-timezone America/Costa_Rica
timedatectl
date
```

------------------------------------------------------------------------

## 3. Instalación de dependencias previas

Antes de instalar Docker debemos contar con los siguientes servicios en
nuestro servidor, los cuales debemos instalar como **Super Usuario**.

``` bash
sudo su

apt install apt-transport-https
apt install ca-certificates
apt install curl
apt install gnupg-agent
apt install software-properties-common
```

------------------------------------------------------------------------

## 4. Agregar la clave GPG oficial de Docker

Copiar tal cual se muestra a continuación en el prompt:

`root@usuario:/home/gnu#`

``` bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

------------------------------------------------------------------------

## 5. Configurar el repositorio estable de Docker

Usamos el siguiente comando para configurar el repositorio estable de la
versión a instalar.

``` bash
echo   "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu   $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

------------------------------------------------------------------------

## 6. Instalar Docker Engine

Actualizamos los repositorios para instalar la última versión de Docker
Engine y del contenedor.

``` bash
apt update
apt-get install docker-ce docker-ce-cli containerd.io docker-compose
```

------------------------------------------------------------------------

## 7. Verificar el servicio Docker

Verificamos que el servicio esté instalado y en ejecución.

``` bash
service docker status
```

**Link de referencia:**\
https://docs.docker.com/engine/install/ubuntu/

------------------------------------------------------------------------

## 8. Descargar la imagen de PostgreSQL

Una vez que tenemos Docker instalado, el siguiente paso es descargar la
imagen que contiene la instalación de PostgreSQL lista para ser
utilizada.

``` bash
docker pull postgres
```

------------------------------------------------------------------------

## 9. Ver detalles de la imagen

``` bash
docker image ls
```

------------------------------------------------------------------------

## 10. Crear y ejecutar el contenedor

Para ejecutar la imagen descargada y crear el contenedor debemos
suministrar:

-   Nombre del contenedor\
-   Contraseña del usuario de la base de datos\
-   Redirección de puerto\
-   Nombre de la imagen

``` bash
docker run --name contenedor-psql -e POSTGRES_PASSWORD=linux -d -p 2022:5432 postgres
```

------------------------------------------------------------------------

## 11. Conectarse al servidor PostgreSQL desde psql

Una vez creado el contenedor de Docker con un servidor PostgreSQL
activo, podemos conectarnos mediante el cliente **psql**.

``` bash
docker run -it --rm --link contenedor-psql:postgres postgres psql -h postgres -U postgres
```

Se introduce la contraseña del usuario y ya podemos trabajar en el
**Sistema Gestor de Bases de Datos**.

------------------------------------------------------------------------

## 12. Ver contenedores en ejecución

``` bash
docker ps
```

------------------------------------------------------------------------

## 13. Detener un contenedor

``` bash
docker stop contenedor-psql
docker ps
```

------------------------------------------------------------------------

## 14. Iniciar un contenedor existente

**Nota:**

La instrucción:

``` bash
docker run --name contenedor-psql -e POSTGRES_PASSWORD=linux -d -p 2022:5432 postgres
```

Se ejecuta **solo una vez**, ya que esta instrucción **crea el
contenedor**.\
Si se intenta ejecutar nuevamente, Docker indicará que **ya existe un
contenedor con ese nombre**.

Para iniciarlo nuevamente utilizamos:

``` bash
docker start NombreDelContenedor
docker ps
```

------------------------------------------------------------------------

## 15. Conexión mediante PgAdmin4

Para finalizar, nos conectamos al contenedor por medio de **PgAdmin4**.

------------------------------------------------------------------------

## 16. Exportar estructura y datos desde PostgreSQL nativo

Nos conectamos a la **máquina virtual con PostgreSQL nativo** y
exportamos:

-   Primero **la estructura**
-   Luego **los datos**

### Pestaña General

-   **Filename:** Ruta donde guardar el respaldo y nombre del archivo\
-   **Format:** Plain\
-   **Encoding:** UTF8

### Pestaña Data Options

-   Only schema / Only data\
-   Blobs

### Pestaña Query Options

-   Use Insert Commands

### Pestaña Table Options

-   Use Column Inserts

------------------------------------------------------------------------

## 17. Cargar respaldo en el contenedor

Para realizar la carga en el contenedor:

1.  Crear una **base de datos nueva**
2.  Abrir la herramienta **Query Tool**
3.  Buscar el archivo con la **estructura** y ejecutar el proceso
4.  Buscar el archivo con los **datos** y ejecutar el proceso

------------------------------------------------------------------------

# Links de referencia

https://todopostgresql.com/docker-con-postgresql-2/

# PARTE II: Crear Contenedor de PgAdmin

------------------------------------------------------------------------

## 1. Ingresar al Servidor - Opcional

Credenciales de acceso:

-   **Usuario:** gnu
-   **Clave:** linux

Luego cambiar a superusuario:

``` bash
sudo su
```

Ingresar la clave cuando sea solicitada.

------------------------------------------------------------------------

## 2. Crear el contenedor de PgAdmin

Ejecutar el siguiente comando para crear el contenedor de **PgAdmin4**
utilizando Docker.

``` bash
docker run -d -p 8080:80 --name nombrecontenedor -e 'PGADMIN_DEFAULT_EMAIL=user@domain.com' -e 'PGADMIN_DEFAULT_PASSWORD=linux' dpage/pgadmin4
```

Este comando realiza lo siguiente:

-   Crea un contenedor llamado **nombrecontenedor**
-   Expone el servicio en el **puerto 8080**
-   Define el **correo de administrador**
-   Define la **contraseña de acceso**
-   Utiliza la imagen oficial **pgAdmin4**

------------------------------------------------------------------------

## 3. Acceder a PgAdmin desde el navegador

Desde la **máquina física**, abrir un navegador web e ingresar la
siguiente dirección:

    http://IP-SERVIDOR:8080

------------------------------------------------------------------------

## 4. Credenciales de acceso

Las credenciales para ingresar son las mismas definidas en el comando de
creación del contenedor.

-   **Usuario:** el correo definido en `PGADMIN_DEFAULT_EMAIL`
-   **Contraseña:** la definida en `PGADMIN_DEFAULT_PASSWORD`

