
## P9: Apache e Virtual Host

### Fago un docker compose para crear as dúas máquinas a a rede compartida entre elas
 
```
# Contenedor de apache:
services:
  web:
    container_name: apache_P9 # nombre 
    image: php:7.4-apache # imaxe
    ports:
    - '8080:80' # le cambio el puerto local por uno libre como el 8080
    - '8000:8000' #porto web
    # mapeamos os directorios:
    volumes:
    - ./www:/var/www/
    - ./confApache/sites-available:/etc/apache2/sites-available
    - ./confApache/sites-enabled:/etc/apache2/sites-enabled
  

    
    dns:
      - 3.190.0.1 # dirección do servidor DNS (o bind9) 
    networks:
      practica_9:
        # ip fija del contenedor
        ipv4_address: 3.190.0.22 # ip fixa
        
  # contenedor servidor de dns - bind9
  bind9:
    container_name: bind9_P9 # nome


    image: ubuntu/bind9


    platform: linux/amd64
    ports:
      - 58:53/tcp
      - 58:53/udp #mapeo do porto 57 do host ao 53 da maquina     
    networks:
      practica_9:
        ipv4_address: 3.190.0.1 # ip fixa
    volumes:
      - ./conf:/etc/bind
      - ./zonas:/var/lib/bind
# red propia
networks:
  practica_9: # nome da rede
    ipam: 
      driver: default
      config:
        - subnet: 3.190.0.0/16
          ip_range: 3.190.0.0/24
          gateway: 3.190.0.254

```

### Creo as carpetas de configuración e zonas do DNS

mkdir zonas conf

**En conf creo o arquivo named.conf:** 

```
zone "aprendendoensri.int" {
	type master;
	file "/var/lib/bind/db.aprendendoensri.int";
	allow-query {
		any;
		};
	};

zone "zonasx.int" {
	type master;
	file "/var/lib/bind/db.zonasx.int";
	allow-query {
		any;
		};
	};
	
	
	
options {
	directory "/var/cache/bind";

	forwarders {
	 	8.8.8.8;
		1.1.1.1;
	 } ;
	 forward only;

	listen-on { any; };
	listen-on-v6 { any; };

	allow-query {
		any;
	};
};

```

**En zonas creo os arquivos de configuración de cada zona:**

zona aprendendoensri.int:
```
$TTL 38400	; 10 hours 40 minutes
@		IN SOA	ns.aprendendoensri.int. some.email.address. (
				17161018   ; serial
				10800      ; refresh (3 hours)
				3600       ; retry (1 hour)
				604800     ; expire (1 week)
				38400      ; minimum (10 hours 40 minutes)
				)
@		IN NS		ns.aprendendoensri.int.
ns		IN A		3.190.0.1
celta		IN A		3.190.0.22
40k		IN A		3.190.0.22
```

Zona zonasx.int:
```
$TTL 38400	; 10 hours 40 minutes
@		IN SOA	ns.zonasx.int. some.email.address. (
				17161019   ; serial
				10800      ; refresh (3 hours)
				3600       ; retry (1 hour)
				604800     ; expire (1 week)
				38400      ; minimum (10 hours 40 minutes)
				)
@		IN NS		ns.zonasx.int.
ns		IN A		3.190.0.1
celta		IN A		3.190.0.22
40k		IN A		3.190.0.22
```

### Creo a carpeta configuración de apache e os arquivos:
mkdir confApache

arquivos: **httpd.conf e mime.types** (este último é porque dábame un erro. Pode ser porque estaba usando a imaxe httpd.)

No httpd.conf configuro algunhas liñas como:
```
#
ServerAdmin lucas@danielcastelao.org

#
# ServerName gives the name and port that the server uses to identify itself.
# This can often be determined automatically, but we recommend you specify
# it explicitly to prevent problems during startup.
#
# If your host doesn't have a registered DNS name, enter its IP address here.
#
ServerName 3.190.0.1

#
```
### Agora probo a ver si o meu servidor funciona usando só o servicio DNS da miña máquina virtual:

Para elo, entro dende o host no arquivo /etc/systemd/resolved.conf e en **RESOLVE** agrego a dirección DNS, que será a dirección do DNS do docker,  3.190.0.1 

![imagen resolve dns](https://github.com/luk295/P9-Apache-e-Virtual-Host/blob/main/resolve-dns.png)

Agora reinicio o servicio con sudo service systemd-resolved restart
### Creando os volumes:

**No meu directorio de traballo, creo as carpetas www e confApache**

-En www, crearei as carpetas 'celta' e '40k'. Onde irán os seus respectivos index.html

-En confApache, creo as carpetas 'sites-available' e 'sites-enabled'. As dúas carpetas ten o mesmo contenido: os arquivos 'celta.conf' e '40k.conf'.

Nestos arquivos, irán os VirtualHost:

celta.conf:
```
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName celta.zonasx.int
    ServerAlias www.celta.zonasx.int
    DocumentRoot /var/www/celta
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

```
40k.cnf:
```                                             
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    ServerName 40k.zonasx.int
    ServerAlias www.40k.zonasx.int
    DocumentRoot /var/www/40k
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

```

Agora que xa teño as carpetas de configuración do Apache creadas na miña máquina, queda mapealas para que se "carguen" na máquina virtual. Para iso no docker úsase os volumes:

```
    volumes:
    - ./www:/var/www/
    - ./confApache/sites-available:/etc/apache2/sites-available
    - ./confApache/sites-enabled:/etc/apache2/sites-enabled
  
```
Agora levanto o Docker Compose e carga coas configuración do Apache. E do dns para que poida resolver os nomes:

### Probando que funciona as dúas páxinas:

![imagen funciona](https://github.com/luk295/P9-Apache-e-Virtual-Host/blob/main/funciona.png)


