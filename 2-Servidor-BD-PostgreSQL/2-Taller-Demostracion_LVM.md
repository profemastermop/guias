# IF5100 Administración de Bases de Datos

**Profesor:** MSc. Andrés Alberto Cortés Fuentes  
**Universidades:** UNA - UTN - UCR Sedes Liberia  

---

# Implementación y Puesta en Producción: LVM para Administrar Almacenamiento - PostgreSQL

Enlace de referencia: https://wadhahdaouehi.tn/2013/10/lvm-mirroring-protect-your-data-from-a-disk-failure/

Realizar un clone completo de la maquina virtual donde tenemos instalado
el PostgreSQL para agregarle dos discos de 15G cada uno.

------------------------------------------------------------------------

## Actualizar zona horaria

Actualizamos la zona horaria para establecer la fecha y hora correcta.

``` bash
date
timedatectl
timedatectl set-timezone America/Costa_Rica
timedatectl
date
```

------------------------------------------------------------------------

## Verificar discos duros

Verificamos los dos discos duros nuevos para crear el Grupo de
Volúmenes:

``` bash
fdisk -l
```

------------------------------------------------------------------------

## Crear Volúmenes Físicos

Creamos los Volúmenes Físicos a partir de cada disco duro.

``` bash
pvs
pvcreate /dev/sdb
pvs
pvcreate /dev/sdc
pvs
```

**Nota:**\
Ya la máquina tenía el **sda**, cuando se añaden más, van en orden
alfabético.

------------------------------------------------------------------------

## Crear Grupo de Volúmenes

``` bash
vgs
vgcreate vg_BaseDatos /dev/sdb /dev/sdc
vgs
```

**Nota:**\
El profesor puede pedir en el examen que utilicen un nombre diferente
para el grupo de volúmenes.

------------------------------------------------------------------------

## Crear Volúmenes Lógicos

Los Volúmenes Lógicos equivalen a las particiones normales en un disco
duro.

``` bash
lvs
lvcreate -n lv_BD_Produccion vg_BaseDatos -L 2G
lvs
lvcreate -n lv_BD_Desarrollo vg_BaseDatos -L 10G
lvs
```

------------------------------------------------------------------------

## Dar formato a los Volúmenes Lógicos

``` bash
lsblk -f
mkfs.ext4 /dev/vg_BaseDatos/lv_BD_Produccion
lsblk -f
mkfs.ext4 /dev/vg_BaseDatos/lv_BD_Desarrollo
lsblk -f
```

------------------------------------------------------------------------

## Configurar montaje automático

Editamos el archivo:

``` bash
vim /etc/fstab
```

Agregamos las siguientes líneas:

``` bash
/dev/vg_BaseDatos/lv_BD_Produccion /BD_Produccion ext4 defaults 0 0
/dev/vg_BaseDatos/lv_BD_Desarrollo /BD_Desarrollo ext4 defaults 0 0
```

------------------------------------------------------------------------

## Crear puntos de montaje

``` bash
cd /

ls -lt
mkdir BD_Produccion
ls -lt
mkdir BD_Desarrollo
ls -lt
```

------------------------------------------------------------------------

## Montar Volúmenes Lógicos

``` bash
df -vh

mount BD_Produccion/

df -vh

mount BD_Desarrollo/
```

Validamos con:

``` bash
df -vh
```

------------------------------------------------------------------------

## Detener PostgreSQL

Detenemos el servicio para cambiar la ruta donde se almacenan las bases
de datos.

``` bash
/etc/init.d/postgresql stop
/etc/init.d/postgresql status
```

------------------------------------------------------------------------

## Mover datos de PostgreSQL

Nos dirigimos a la ruta por defecto:

``` bash
cd /var/lib/postgresql/16/
ls -ltr
du -sh main
```

Copiamos la carpeta **main** al nuevo volumen lógico.

``` bash
rsync -vazul --progress main /BD_Produccion/
```

**Recomendación:**\
Podemos abrir otra terminal y verificar que el directorio está vacío
antes de copiar.

**Nota alternativa:**

``` bash
scp -r main /BD_Produccion/
```

Esta opción copia la carpeta pero asigna permisos del usuario **root**.\
Luego habría que ejecutar:

``` bash
chown postgres.postgres -R
chmod 700 main -R
```

------------------------------------------------------------------------

## Verificar copia

``` bash
cd /BD_Produccion
ls -ltr
du -sh main/
```

------------------------------------------------------------------------

## Cambiar ruta de almacenamiento en PostgreSQL

Editar el archivo:

``` bash
vim /etc/postgresql/16/main/postgresql.conf
```

Mostrar números de línea:

    :se nu

La línea **42** debe quedar así:

    data_directory = '/BD_Produccion/main'

------------------------------------------------------------------------

## Iniciar nuevamente PostgreSQL

``` bash
/etc/init.d/postgresql start
/etc/init.d/postgresql status
```

------------------------------------------------------------------------

## Verificar nuevo almacenamiento

``` bash
cd /BD_Produccion/main/base/
```

Ejecutar:

``` bash
while true; do ls -lt; sleep 3; clear; done;
```

Para salir:

    Ctrl + C

Crear una base de datos nueva desde **PgAdmin** y observar cómo aparece
una carpeta nueva correspondiente a la base de datos recién creada.

------------------------------------------------------------------------

# Ampliación de Volumen Lógico

Ahora vamos a asumir que nuestro volumen lógico **lv_BD_Produccion**
está por llenarse y necesitamos ampliar su tamaño sin bajar el servicio
o el servidor.

En el escenario de demostración agregamos dos discos de **15GB** cada
uno.

Ambos discos se agregan al grupo de volúmenes **vg_BaseDatos**.

Creamos dos volúmenes lógicos:

-   **lv_BD_Produccion** de 2GB
-   **lv_BD_Desarrollo** de 10GB

Quedan **18GB libres** en el grupo de volúmenes, que utilizaremos para
ampliar **lv_BD_Produccion**.

------------------------------------------------------------------------

## Ampliar el volumen lógico

``` bash
df -vh
lvs
lvextend -L +13Gb /dev/vg_BaseDatos/lv_BD_Produccion
lvs
df -vh
resize2fs /dev/vg_BaseDatos/lv_BD_Produccion
lvs
df -vh
```

