# Instructivo de Configuración de NFS (Network File System)

**Hecho Por:** MSc. Andrés Alberto Cortés Fuentes  
**Universidades:** UNA - UTN - UCR Sedes Chorotega Liberia

---

## Servicio para Compartir Archivos entre Sistemas GNU/Linux

Este instructivo describe el proceso de instalación y configuración del servicio **NFS** para compartir archivos entre sistemas GNU/Linux (Servidor y Cliente).

---

## 1. Configuración del Servidor

### Instalación del Servicio

```bash
apt install nfs-kernel-server
```

### Configuración del Archivo `/etc/exports`

Editar el archivo `vim /etc/exports` y agregar la siguiente línea:

```bash
/datos *(rw,fsid=1,no_subtree_check)
```

### Reiniciar el Servicio

```bash
/etc/init.d/nfs-kernel-server restart
```

---

## 2. Configuración del Cliente

### Recomendaciones

Instalar los siguientes paquetes: vim es el editor que hemos utilizado a lo largo del curso, openssh-server para poder establecer conexiones remotas al cliente y realizar tareas de configuración desde el MobaXterm.

Que el equipo cliente cuente con una interfaz en modo Red Interna y otra en modo Solo Anfitrión (esta última para conectarse desde el MobaXterm)


```bash
apt-get install vim
```

```bash
apt-get install openssh-server
```


### Instalación de Paquetes del Servicio NFS

```bash
apt-get install nfs-common
```

### Configuración del Archivo `/etc/fstab`

Editar el archivo `vim /etc/fstab` y agregar al final la siguiente línea:

```bash
192.168.100.1:/datos /disco_datos nfs4 _netdev,noauto,timeo=14 0 0
```

---

## 3. Montaje del Recurso Compartido

Desde la raíz `/`, crear la carpeta donde se montará el recurso compartido:

```bash
mkdir /disco_datos
chmod 777 /disco_datos -R
```

Finalmente, ejecutar el siguiente comando para montar el recurso compartido:

```bash
mount /disco_datos
```

---

✅ **Con esto el cliente GNU/Linux tendrá acceso al recurso compartido `/datos` del servidor mediante NFS.**
