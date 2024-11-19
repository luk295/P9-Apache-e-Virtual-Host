
## P9: Apache e Virtual Host

### Fago un docker compose para crear as dúas máquinas a a rede compartida entre elas
 
```
# Contenedor de apache:
services:
  web:
    container_name: apache_P9 # nombre 
    image: httpd:latest # imaxe
    ports:
      - "80:80" #porto web
    # mapeamos el directorio raiz
    volumes:
      - ./paginas:/usr/local/apache2/htdocs
      - ./confApache:/usr/local/apache2/conf
    
    dns:
      - 173.190.0.1 # dirección do servidor DNS (o bind9) 
    networks:
      practica_9:
        # ip fija del contenedor
        ipv4_address: 173.190.0.22 # ip fixa
        
  # contenedor servidor de dns - bind9
  bind9:
    container_name: bind9_P9 # nome


    image: ubuntu/bind9


    platform: linux/amd64
    ports:
      - 57:53 #mapeo do porto 57 do host ao 53 da maquina
    networks:
      practica_9:
        ipv4_address: 173.190.0.1 # ip fixa
    volumes:
      - ./conf:/etc/bind
      - ./zonas:/var/lib/bind
      
# red propia
networks:
  practica_9: # nome da rede
    ipam: 
      driver: default
      config:
        - subnet: 173.190.0.0/16
          gateway: 173.190.0.1
```
