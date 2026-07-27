# Instructivo: Curso Administración de Servidores con Software Libre

**Hecho Por:** MSc. Andrés Alberto Cortés Fuentes  
**Universidades:** UNA - UTN - UCR Sedes Liberia

# Guía de Configuración Servicio DHCP IPv4 (Ubuntu Server)

## 1. Instalación del Servicio
Instalamos el servicio del DHCP con la instrucción:
```bash
apt install isc-dhcp-server
```

## 2. Configuración de la Interfaz de Red
Una vez instalado el servicio, le indicamos al sistema por cuál tarjeta de red va a escuchar las peticiones ingresando al archivo:
```bash
vim /etc/default/isc-dhcp-server
```
Dentro de este archivo, busque la línea de `INTERFACESv4` y especifique su interfaz de red (por ejemplo, `enp0s8`):
```text
INTERFACESv4="enp0s8"
```
*(Nota: Las líneas que inician con el símbolo `#` son comentarios).*

## 3. Configuración Principal del DHCP
Nos movemos al directorio de configuración del DHCP con la instrucción:
```bash
cd /etc/dhcp
```
Dentro de este directorio editaremos el archivo principal:
```bash
vim dhcpd.conf
```

### 3.1. Modificaciones Iniciales
En el archivo `dhcpd.conf`, realice los siguientes cambios:
1. Reemplace el nombre de dominio. Busque:
   `option domain-name "example.org";`
   Y reemplácela por:
   `option domain-name "gnulinux.mop";`
2. Comente las siguientes líneas (agregando `#` al inicio):
   ```text
   #option domain-name-servers ns1.example.org, ns2.example.org;
   #default-lease-time 600;
   #max-lease-time 7200;
   ```

### 3.2. Definición de Segmentos de Red y Reservas
Agregue los siguientes parámetros al final del archivo para definir los diferentes segmentos de red (útil si se tiene una segmentación lógica a nivel de VLANs) y las reservas de direcciones por MAC:

```conf
#Aca especificamos la direccion ip del servidor
local-address 192.168.100.1;

#Directriz que especifica la definicion de los segmentos de red
shared-network curso
{
    #Definicion de segmento en su nivel mas basico
    subnet 192.168.0.0 netmask 255.255.255.0 
    {
    }
    
    #Definicion de segmento de proposito general
    subnet 192.168.100.0 netmask 255.255.255.0
    {
        authoritative;
        default-lease-time 36000;
        max-lease-time 72000;

        #Rango de direcciones, a asignar en caso de que el equipo conectado a la red no tenga una reserva asociada.
        #range dynamic-bootp 192.168.100.200 192.168.100.253;
        
        #Aca especificamos la direccion ip de nuestro servidor de salida a internet o sea nuestro gateway
        option routers 192.168.100.3;
        option subnet-mask 255.255.255.0;
        option broadcast-address 192.168.100.255;
        
        #Aca especificamos la direccion ip de nuestro servidor NIS en caso de que contemos con este servicio
        option nis-servers 192.168.100.1;
        option nis-domain "gnulinux.mop";
        
        #Aca especificamos la lista de servidores DNS en el orden que tengamos definido en nuestra red.
        option domain-name-servers 192.168.100.1, 8.8.8.8, 8.8.4.4;
        allow unknown-clients;

        #Aca para los siguientes parametros definimos la direccion ip del servidor.
        option netbios-name-servers 192.168.100.1;
        option netbios-dd-server 192.168.100.1;
        option netbios-node-type 8;
        option netbios-scope "";
        option finger-server 192.168.100.1;
        option log-servers 192.168.100.1;
        option x-display-manager 192.168.100.1;
        
        get-lease-hostnames true;
        use-host-decl-names true;
    } # Cierra subnet
} # Cierra share-network

#A continuacion reserva de direcciones
host nombre_cliente.gnulinux.mop
{
    hardware ethernet 00:0C:29:18:7F:21;
    fixed-address 192.168.100.10;
}
```

## 4. Reinicio y Monitoreo del Servicio
Una vez especificados estos valores, como root reiniciamos el servicio DHCP. Puede utilizar cualquiera de las siguientes opciones:
```bash
/etc/init.d/isc-dhcp-server restart
```
O bien deteniéndolo e iniciándolo nuevamente:
```bash
/etc/init.d/isc-dhcp-server stop
/etc/init.d/isc-dhcp-server start
```
*(También se puede usar `service isc-dhcp-server stop` y `start`).*

Para poder ver el estado del servicio DHCP. utilizamos el siguiente comando:
```bash
/etc/init.d/isc-dhcp-server status
```

Para monitorear el servicio, es recomendable tener abierta otra terminal ejecutando el comando:
```bash
tail -f /var/log/syslog
```

## 5. Configuración en los Clientes
El trabajo restante se realiza del lado de los clientes:
1. Recopilar las direcciones MAC de las computadoras para registrarlas en las reservas de `dhcpd.conf` (en la sección `hardware ethernet`).
2. Configurar la tarjeta de red en modo **Red Interna**.
3. Verificar los datos IP que el Servidor DHCP asigne al equipo.