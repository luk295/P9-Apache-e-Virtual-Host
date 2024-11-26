
## P9: Apache e Virtual Host

### Fago un docker compose para crear as dúas máquinas a a rede compartida entre elas
 
```
# Contenedor de apache:
services:
  web:
    container_name: apache_P9 # nombre 
    image: httpd:latest # imaxe
    ports:
    - '8080:80' # le cambio el puerto local por uno libre como el 8080
    - '8000:8000' #porto web
    # mapeamos el directorio raiz
    volumes:
      - ./paginas:/usr/local/apache2/htdocs
      - ./confApache:/usr/local/apache2/conf
    
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

mkdir zonas confi

* En conf creo o arquivo named.conf: *

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

** En zonas cero os arquivos de configuración de cada zona: **

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
@		IN NS	ns
ns		IN A		3.190.0.1
www		IN A		3.190.0.50
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
@		IN NS	ns	ns.zonasx.int.
ns		IN A		3.190.0.2
www		IN A		3.190.0.50
```

### Creo a carpeta configuración de apache e os arquivos:
mkdir confApache

arquivos: **httpd.conf e mime.types**

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

(captura configurada reslve-dns.png).




