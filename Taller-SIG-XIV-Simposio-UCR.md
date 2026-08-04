# XIV Simposio de Informática Empresarial
## Taller: Implementación de Infraestructura de Datos Espaciales con Software Libre
**Profesor:** MSc. Andrés Alberto Cortés Fuentes  
**Fecha:** Agosto 2026  

---

### 1era Parte: Instalación VirtualBox

- Crear punto de restauración en Windows
- Deshabilitar Firewall
- Deshabilitar Antivirus
- Proceder con la instalación de VirtualBox...

#### Creación máquina virtual
- Dos interfaces de red: NAT y Adaptador Solo Anfitrión (Modo promiscuo Permitir Todo)
- 2GB RAM
- 80GB Disco Duro

#### Instalación Ubuntu Server
- Modo guiado
- Conexión por SSH a través de MobaXterm

---

### 2da Parte: Instalación Docker

#### Instalación de dependencias
Antes de instalar Docker debemos contar con los siguientes servicios en nuestro servidor, los cuales debemos instalar como Super Usuario.

```bash
sudo su

apt install apt-transport-https
apt install ca-certificates
apt install curl
apt install gnupg-agent
apt install software-properties-common
```

#### Agregar la clave GPG oficial de Docker
Copiar tal cual se muestra a continuación en el prompt:

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

#### Configurar el repositorio estable de Docker
Usamos el siguiente comando para configurar el repositorio estable de la versión a instalar.

```bash
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

#### Instalar Docker Engine
Actualizamos los repositorios para instalar la última versión de Docker Engine y del contenedor.

```bash
apt update
apt-get install docker-ce docker-ce-cli containerd.io docker-compose
```

#### Verificar el servicio Docker
Verificamos que el servicio esté instalado y en ejecución.

```bash
service docker status
```

**Link de referencia:** [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)

---

### 3ra Parte: Creación de Contenedor

#### Descarga de la Imagen
```bash
docker pull postgis/postgis:latest
```

#### Creación del contenedor
```bash
docker run --name sig-psql -e POSTGRES_PASSWORD=linux -d -p 2022:5432 postgis/postgis:latest
```

#### Ingreso a la consola psql del contenedor
```bash
docker run -it --rm --link sig-psql:postgres postgis/postgis:latest psql -h postgres -U postgres
```

#### Salimos de la consola psql
```bash
Ctrl + D
```

---

### 4ta Parte: Transferencia de archivos a través de PowerShell y MobaXterm

Descargar la carpeta titulada: `Taller-SIG-Simposio-UCR` del siguiente enlace de Google Drive: [https://drive.google.com/drive/folders/1KaJqBPwr2Rxd00bouDVNVTXbiNLSD-ls?usp=sharing](https://drive.google.com/drive/folders/1KaJqBPwr2Rxd00bouDVNVTXbiNLSD-ls?usp=sharing)

#### Copiar de Windows a Ubuntu Server
```bash
scp -r "C:\Users\Vane\Downloads\Taller-SIG-Simposio-UCR" gnu@192.168.56.X:/tmp/
```

#### Copiar de Ubuntu Server al Contenedor
```bash
docker cp /tmp/Taller-SIG-Simposio-UCR sig-psql:/opt/
```

> **Nota:** ¡Estos comandos no se ejecutan, es un recurso adicional para tener en cuenta!

#### Copiar del Contenedor a Ubuntu Server
```bash
docker cp sig-psql:/opt/Taller-SIG-Simposio-UCR/limitecantonal_5k/cantones.sql /tmp/
```

#### Copiar desde Ubuntu Server a Windows
```bash
scp -r gnu@192.168.56.X:/tmp/cantones.sql "C:\Users\Vane\Downloads\"
```

---

### 5ta Parte: Creación Base de Datos y proceso de importación a partir de archivos .SHP

Creamos nuestra Base de Datos y la extensión geoespacial.

#### Ingreso a la consola psql del contenedor
```bash
docker run -it --rm --link sig-psql:postgres postgis/postgis:latest psql -h postgres -U postgres
```

```sql
CREATE DATABASE bdsig;
\c bdsig
CREATE EXTENSION postgis;
```

#### Salimos de la consola psql
```bash
Ctrl + D
```

#### Nos ambientamos en la terminal del contenedor:
```bash
docker exec -it sig-psql bash
```

#### Actualizamos los repositorios del contenedor:
```bash
apt update
```

#### Instalamos herramientas adicionales de postgis (por ejemplo shp2pgsql):
```bash
apt-get install -y postgis
```

#### Ahora, siempre desde dentro del contenedor, nos ubicamos en el directorio
Que contiene la información que copiamos previamente desde Windows a Ubuntu y de Ubuntu al contenedor.
```bash
cd /opt/Taller-SIG-Simposio-UCR/limiteprovincial_5k
ls -lt
```

#### Comando para crear SQL e importar en la Base de Datos a partir del archivo SHP
```bash
shp2pgsql -I -s 5367 -W LATIN1 limiteprovincial_5k.shp public.provincias | psql -U postgres -d bdsig
```

---

### 6ta Parte: Demostración desde la aplicación de Escritorio Qgis 
Despliegue de la información gráfica almacenada en la Base de Datos.

**Configuración Parámetros de Conexión:**
- **Nombre:** taller-sig
- **Servicio:** (vacío)
- **Anfitrión:** Ip del Servidor con Ubuntu Server - `192.168.56.X`
- **Puerto:** `2022`
- **Base de Datos:** bdsig
- **Modo SSL:** preferir
- **Session ROLE:** (vacío)

**Apartado Autenticación / Pestaña Básica:**
- **Nombre de Usuario:** postgres
- **Contraseña:** linux

---

### 7ma Parte: Reto para los participantes!!!