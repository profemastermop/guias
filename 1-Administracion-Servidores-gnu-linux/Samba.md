# Instructivo de Instalación y Configuración de SAMBA

**Hecho Por:** MSc. Andrés Alberto Cortés Fuentes  
**Universidades:** UNA - UTN - UCR Sedes Chorotega Liberia  

---

## Instalación del Servicio

```bash
apt-get install samba
```

**Lectura recomendada:**  
[Permisos y Propietarios en Linux](https://www.hostinger.com/es/tutoriales/cambiar-permisos-y-propietarios-linux-linea-de-comandos)

---

## Creación de Directorios y Usuarios

En la partición `/datos` vamos a crear una carpeta llamada `alumnos` y dentro de esta crearemos las siguientes carpetas:  
`andres`, `juan` y `pedro`.

Antes de ello, debemos crear los usuarios correspondientes definiendo la ruta del directorio home de la siguiente manera:  
`/datos/alumnos/nombre`

Pero antes creamos el grupo **curso_linux** con el GUID = `9000`.

Vemos el archivo donde se almacenan los grupos antes de la creación

```bash
cat /etc/group
```

Comando para crear grupos

```bash
groupadd -g 9000 curso_linux
```

Vemos el archivo donde se almacenan los grupos despúes de la creación

```bash
cat /etc/group
```

Vemos el archivo donde se almacenan los usuarios antes de la creación

```bash
cat /etc/passwd
```

Comandos para crear usuarios

```bash
useradd -u 9001 -d /datos/alumnos/andres -g curso_linux andres
useradd -u 9002 -d /datos/alumnos/juan -g curso_linux juan
useradd -u 9003 -d /datos/alumnos/pedro -g curso_linux pedro
```

Vemos el archivo donde se almacenan los usuarios despúes de la creación

```bash
cat /etc/passwd
```

Vemos el archivo donde se almacenan las contraseñas antes de la creación

```bash
cat /etc/shadow
```

Creamos las contraseñas para cada usuario

```bash
passwd andres
passwd juan
passwd pedro
```

Vemos el archivo donde se almacenan las contraseñas despúes de la creación

```bash
cat /etc/shadow
```

Asignamos los permisos y dueños a cada carpeta, previo a ello nos cambiamos de directorio:

```bash
cd /datos/alumnos
```

Vemos los permisos asignados antes

```bash
ls -lt
```

```bash
chown andres:curso_linux /datos/alumnos/andres
chown juan:curso_linux /datos/alumnos/juan
chown pedro:curso_linux /datos/alumnos/pedro

chmod 700 /datos/alumnos/andres
chmod 700 /datos/alumnos/juan
chmod 700 /datos/alumnos/pedro
```

Vemos los permisos asignados despúes

```bash
ls -lt
```

---

## Configuración de Recursos Compartidos

Editar el archivo de configuración `vim /etc/samba/smb.conf` y agregar las siguientes secciones, al final del archivo:

```bash
[andres]
browseable = no
create mode = 700
directory mode = 700
path = /datos/alumnos/andres
write list = andres

[Departamento_TI]
browseable = no
comment = Departamento_TI
create mode = 777
directory mode = 777
path = /datos/departamentos/dpto_ti
write list = @curso_linux
```

---

## Reinicio del Servicio SAMBA

Reiniciar los siguientes servicios para aplicar los cambios:

```bash
/etc/init.d/smbd restart
```

```bash
/etc/init.d/nmbd restart
```

---

## Importar Usuarios de UNIX a SAMBA

```bash
smbpasswd -a nombre_usuario
```

Repetir el comando anterior para cada usuario creado.

---

## Conexión desde Windows

En el cliente Windows se deben crear las unidades de red utilizando alguna de las siguientes rutas:

```
\\nombre_del_equipo_+_dominio\nombre_recurso_compartido
```

o bien:

```
\\ip_servidor\nombre_recurso_compartido
```

## Conexión desde Gnu/Linux

En el cliente gnu/linux se deben crear los accesos a la red utilizando alguna de las siguientes rutas:

```
smb://nombre_del_equipo_+_dominio\nombre_recurso_compartido
```

o bien:

```
smb://ip_servidor\nombre_recurso_compartido
```

---

✅ **Con esto se completa la instalación y configuración básica del servicio SAMBA.**