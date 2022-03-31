# Servidor Web Apache

## Docker Compose

*Contenedor del dns
~~~
  asir_bind9:
    image: internetsystemsconsortium/bind9:9.16
    ports:
      -53:53
    volumes:
      -dns_conf:/etc/bind
    networks:
      red01: Aquí está la IP que tendrá en la red creada: red01
        ipv4_address: 10.1.0.254
~~~
*Cliente con la imagen kasmweb/desktop
~~~
  asir_cliente:
    image: kasmweb/desktop:1.10.0-rolling
    ports: Aquí tiene vinculado dos puertos
      -6901:6901
~~~
* Establecemos la contraseña al usuario del contenedor
~~~
    environment:
      VNC_PW: password
    networks:
      -red01
    dns:
      -10.1.0.254
    stdin_open: true  # docker run -i
    tty: true         # docker run -t
~~~
*Servidor web apache
~~~
  asir_webb:
    image: httpd:latest
    ports:
      -"80:80"
    networks:
      red01:
        ipv4_address: 10.1.0.15
    volumes:
      -apache_index:/usr/local/apache2/htdocs
      -apache_conf:/usr/local/apache2/conf
~~~
* Creamos el contenedor del wireshark 
~~~
asir_wireshark:
    image: linuxserver/wireshark
    cap_add:  
      - NET_ADMIN
    network_mode: host  
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
    ports:
      - 3000:3000
~~~
- Para entrar en wireshark desde el navegador, localhost:3000
*Para finalizar todos los contenedores existentes con external para indicar que ya han sido creados
~~~
networks:
 red01:
  external: true
volumes:
  dns_conf:
    external: true
  apache_conf:
    external: true
  apache_index:
    external: true
~~~

## Configuración maquinas   

*En servidor web

- En el servidor apache crearemos carpetas con sus index para crear las páginas web

- Buscamos la configuracion del volumen del apache y descomentamos esta linea: Include conf/extra/httpd-vhosts.conf y la descomentamos.

- En el directorio extra buscamos el archivo: httpd-vhosts.conf y creamos dos virtual host para cada pagina.
~~~
<VirtualHost *:80>
    ServerAdmin webmaster@dummy-host.example.com
    DocumentRoot "/usr/local/apache2/htdocs/adios"
    ServerName adios.ejemplo.com
    ErrorLog "logs/dummy-host.example.com-error_log"
    CustomLog "logs/dummy-host.example.com-access_log" common
</VirtualHost>

<VirtualHost *:80>
    ServerAdmin webmaster@dummy-host2.example.com
    DocumentRoot "/usr/local/apache2/htdocs/hola"
    ServerName hola.ejemplo.com
    ErrorLog "logs/dummy-host2.example.com-error_log"
    CustomLog "logs/dummy-host2.example.com-access_log" common
</VirtualHost>

 - ServerAdmin: es un contacto al administrador o otra persona
 - DocumentRoot: es la ruta donde estara la pagina web
 - ServerName: Es el nombre de dominio del virtual host
 - ErrorLog: es donde se van a almacenar los logs de errores
 - CustomLog: Es donde se va a almacenar los logs de acceso
~~~

*En Servidor dns

- Creamos una zona dns y agregamos estos cnames:
~~~
hola IN CNAME ejemplo
adios IN CNAME ejemplo
~~~

- Reiniciamos los equipos

*En el cliente

- Vamos al navegador y buscamos primero hola.ejemplo.com y nos aparece el index creado a la pagina hola.

- Despues buscamos adios.ejemplo.com y nos tendrá que aparecer la otra pagina.


## Protocolo HTTPS

~~~
Ahora ponemos el comando para crear las claves y si necesitamos que se creen los archivos en una ruta especifica le pondriamos el path:
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt

Ahora usariampos el comando cp .key o .crt contenedor:ruta-volumen:

sudo docker cp apache-selfsigned.key asir_webb:/usr/local/apache2/conf

docker cp apache-selfsigned.crt asir_webb:/usr/local/apache2/conf

A continuacion inspecionamos el volumen y entramos en el archivo httpd-ssl.conf y en required modules vamos copiando y a traves del httpd.conf las lineas que descomentamos para poner en funcionamiento el htpps.

~~~

~~~
Configuramos de esta forma el virtual host:

<VirtualHost _default_:443>
DocumentRoot "/usr/local/apache2/htdocs/adios"
ServerName adios.prueba.com
ServerAdmin you@example.com
ErrorLog /proc/self/fd/2
TransferLog /proc/self/fd/1
SSLEngine on
SSLCertificateFile "/usr/local/apache2/conf/server.crt"
SSLCertificateKeyFile "/usr/local/apache2/conf/server.key"

~~~

## Wireshark

~~~
Al incluir esto en el docker compose ya tendríiamos creado el contenedor de wireshark, para acceder al wireshark solo deberemos acceder al navegador y poner localhost:3000 y ya podremos escanear los paquetes de la red.
asir_wireshark:
    image: linuxserver/wireshark
    cap_add:  
      - NET_ADMIN
    network_mode: host  
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
    ports:
      - 3000:3000 #optional

Y ya podriamos captar paquetes dns, http y https ademas de comprobar que los httpcontienen texto plano y los https van cifrados.
~~~
