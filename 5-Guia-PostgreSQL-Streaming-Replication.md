# IF5100 Administración de Bases de Datos

**Profesor:** MSc. Andrés Alberto Cortés Fuentes  
**Universidades:** UNA - UTN - UCR Sedes Liberia  

---

# PostgreSQL - Streaming Replication - Ubuntu Server

## Enlaces de referencia

### Ubuntu 20.04
- https://ubuntu.com/server/docs/databases-postgresql
- https://translate.google.com/translate?hl=es&sl=en&u=https://ubuntu.com/server/docs/databases-postgresql&prev=search&pto=aue
- https://www.digitalocean.com/community/tutorials/how-to-set-up-physical-streaming-replication-with-postgresql-12-on-ubuntu-20-04

### Ubuntu 22.04
- https://www.server-world.info/en/note?os=Ubuntu_22.04&p=postgresql&f=3

---

# Configurar tarjeta de red en Ubuntu Server

```bash
sudo su

vim /etc/netplan/50-cloud-init.yaml
```

```yaml
enp0s8:
  dhcp4: false
  addresses: [192.168.56.190/24]
```

## Reiniciar el servicio

```bash
netplan apply
```

---

# PASOS Y CONFIGURACION DE PARAMETROS PARA ESTABLECER LA REPLICACION

# REQUERIMIENTOS TECNICOS

a- Dos máquinas virtuales con Ubuntu Server y el servicio de PostgreSQL instalado localmente.

b- RECOMENDACIÓN: En cada servidor crear un contenedor con PgAdmin para establecer la conexión e ir validando todo lo que se realiza a nivel de la consola. De esta forma en el navegador van contar con dos instancias cada una conectada a un servidor en específico.

---

# ESTO LO REALIZAMOS EN EL SERVIDOR MAESTRO

## 1- Crear usuario de replicación

Primero, debemos crear un usuario de replicación en el SERVIDOR MAESTRO para realizar la conexión con el SERVIDOR REPLICA.

### Ver lista de usuarios

```bash
sudo -u postgres psql -c "\du"
```

### Crear usuario replicador

```bash
sudo -u postgres createuser --replication -P -e replicator
```

Le asignamos la contraseña:

```text
linux
```

### Verificamos nuevamente

```bash
sudo -u postgres psql -c "\du"
```

---

## 2- Configurar parámetros del servidor maestro

Editar el archivo:

```bash
vim /etc/postgresql/16/main/postgresql.conf
```

Mostrar números de línea:

```vim
:se nu
```

Agregar o modificar:

```ini
listen_addresses = '*'
wal_level = replica
max_wal_senders = 10
```

---

## 3- Configurar pg_hba.conf

Editar el archivo:

```bash
vim /etc/postgresql/16/main/pg_hba.conf
```

Agregar la siguiente línea al final del archivo:

```ini
host  replication   replicator   <IP address of the replication>/24      md5
```

---

## 4- Reiniciar el servicio PostgreSQL

```bash
sudo systemctl restart postgresql
```

o

```bash
/etc/init.d/postgresql restart
```

---

# ESTO LO REALIZAMOS EN EL SERVIDOR REPLICA

## 5- Detener el servicio PostgreSQL

```bash
sudo systemctl stop postgresql
```

o

```bash
/etc/init.d/postgresql stop
```

---

## 6- Configurar hot_standby

Editar el archivo:

```bash
vim /etc/postgresql/16/main/postgresql.conf
```

Agregar o modificar:

```ini
hot_standby = on
```

---

## 7- Realizar copia de seguridad y sincronización

### Cambiarnos al usuario postgres

```bash
su - postgres
```

### Cambiarnos al directorio correspondiente

```bash
cd /var/lib/postgresql/16/
```

### Ver contenido del directorio

```bash
ls -lt
```

### Crear respaldo del directorio main

```bash
cp -R /var/lib/postgresql/16/main /var/lib/postgresql/16/main_bak
```

### Verificar contenido nuevamente

```bash
ls -lt
```

### Ver tamaño de directorios

```bash
du -sh *
```

### Eliminar archivos del directorio main

```bash
rm -rf /var/lib/postgresql/16/main/*
```

### Verificar nuevamente tamaños

```bash
du -sh *
```

### Ejecutar respaldo desde el servidor maestro

```bash
pg_basebackup -h <IP adrress of the master> -D /var/lib/postgresql/16/main -U replicator -P -v -R
```

### Significado de las banderas de pg_basebackup

- `-h`: nombre de host o dirección IP del servidor maestro
- `-D`: directorio de datos
- `-U`: usuario utilizado en la operación
- `-P`: muestra reportes de progreso
- `-v`: habilita modo detallado
- `-R`: crea el archivo `standby.signal` y agrega configuraciones en `postgresql.auto.conf`

---

## 8- Iniciar el servicio PostgreSQL en el servidor replica

Nota: primero debemos salir del usuario postgres y regresar a ROOT.

```bash
sudo systemctl start postgresql
```

o

```bash
/ect/init.d/postgresql start
```

---

# ESTO LO REALIZAMOS EN EL SERVIDOR MAESTRO

## 9- Verificar replicación

```bash
sudo -u postgres psql -c "select * from pg_stat_replication;"
```

---

## 10- Configurar replicación sincrónica

Editar el archivo:

```bash
vim /etc/postgresql/16/main/postgresql.conf
```

Agregar o habilitar:

```ini
synchronous_commit = on
synchronous_standby_names = '*'
```

---

## 11- Reiniciar PostgreSQL

```bash
sudo systemctl restart postgresql
```

o

```bash
/etc/init.d/postgresql restart
```

---

## 12- Verificar sincronización

```bash
sudo -u postgres psql -c "select * from pg_stat_replication;"
```

El campo `sync_state` debe cambiar de `async` a `sync`.

---

## 13- Prueba final de replicación

### 13.1- En el SERVIDOR MAESTRO

```bash
sudo -u postgres createdb replica
```

### 13.2- En el SERVIDOR REPLICA

```bash
sudo -u postgres psql -c "\l"
```

### 13.3- Resultado esperado

La base de datos creada en el MAESTRO debe visualizarse en el SERVIDOR REPLICA.