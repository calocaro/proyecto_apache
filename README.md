# ProyectoApache

## Imágenes necesarias

Para nuestro contenedor de un servidor **Apache** con soporte de **PHP** debemos acudir a la documentación de docker donde se nos ofrece dicho contenedor de [Servidor Apache + PHP](https://hub.docker.com/_/php/).

## Configuración Docker-compose

En este caso para levantar el servicio utilizaremos el fichero de [```docker-compose.yml```](https://github.com/calocaro/proyecto_apache/blob/master/docker-compose.yml) donde en el indicaremos:

* **Services**

Donde indicamos el:

- Nombre de nuestro servicio:

```
services:
  apache:
```
- Añadimos el nombre de nuestro contenedor ```container_name```:

```
services:
  apache:
    container_name: apachePHP
```
- A continuación la imagen del contenedor  ```image``` incluyendo su versión:

```
services:
  apache:
    container_name: apachePHP
    image: php:7.2-apache
```
- Los puertos en este caso TPC y UDP con el parámetro ```ports```:

```
services:
  apache:
    container_name: apachePHP
    image: php:7.2-apache
    ports:
      - 80:80/udp
      - 80:80/tcp
```

## Creación de sitios

Para levantar las diferentes páginas html/php debemos crear un nuevo o nuevos directorios para los sitios, para este primer sitio únicamente creamos uno denominado [**Sitio1**](https://github.com/calocaro/proyecto_apache/tree/master/sitio1).

En el directorio añadimos un [**Hola Mundo**](https://github.com/calocaro/proyecto_apache/blob/master/sitio1/index.php) en PHP para realizar la prueba.

## Mapeo directorio sitios

Para mapear nuestro sitio para utilizarlo en el servidor de Apache, debemos especificarlo en el [```docker-compose.yml```](https://github.com/carocalo/proyecto_apache/blob/master/docker-compose.yml) en el apatardo de ```volumes: ```, indicando que nuestro directorio [**Sitio1**](https://github.com/calocaro/proyecto_apache/tree/master/sitio1) se corresponde con el ```/var/www/html``` de Apache.

```
services:
  apache:
    container_name: apachePHP
    image: php:7.2-apache
    ports:
      - 80:80/udp
      - 80:80/tcp
    volumes:
      - /home/proyectoApache/sitio1:/var/www/html
```

De esta manera si levantamos el servicio ejecutando en el terminal ```docker-compose up``` podremos visualizar desde nuestro buscador el  [**localhost**](http://localhost/) con el [**Hola Mundo**](https://github.com/calocaro/proyecto_apache/blob/master/sitio1/index.php) que hemos indicado previamente.


_![GIF](https://github.com/calocaro/proyecto_apache/blob/master/pictures/mOFSQJB.gif)_

## Creación sitios virtuales

Nosotros queremos modificar la configuración de nuestro servidor Apache, para ello debemos acceder al fichero de configuración del mismo y mapearlo en nuestro local. Para ello iremos volumen de Apache, donde se encontará su fichero de configuración.

*Primeramente abrimos dicho volumen en una pestaña aparte*

_![Imagen](https://github.com/calocaro/proyecto_apache/blob/master/pictures/volumenIndependiente.png)_

Una vez abierto seleccionaremos todo el contenido de los ficheros de configuración, el cual "pegaremos" en una carpeta nueva que creemos en nuestro directorio del Proyecto apache denominado [**confApache**](https://github.com/calocaro/proyecto_apache).

Como vamos a mapear nuestro directorio [```confApache```](https://github.com/calocaro/proyecto_apache/tree/master/confApache) al directorio de configuración del servidor Apache en el volumen, debemos actualizar el [```docker-compose.yml```](https://github.com/calocaro/proyecto_apache/blob/master/docker-compose.yml).
    
```yml
    volumes:
      - /home/proyecto_apache/sitio1:/var/www/html
      - ./confApache:/etc/apache2

```

Crearemos un nuevo sitio como indicamos en el apartado de creación de sitios, lo denominaremos [**Sitio2**](https://github.com/calocaro/proyecto_apache/tree/master/html/sitio2) para posteriormente poder mapearlo con el anterior y no independientemente, por lo que debemos modificar el [```docker-compose.yml```](https://github.com/calocaro/proyecto_apache/blob/master/docker-compose.yml) añadiendo el directorio principal de sitios ```html``` donde se encuentra el **sitio1** y el **sitio2**.

```yml
    volumes:
      - ./html:/var/www/html

```
Posteriormente accedemos al fichero de configuración de los [```ports.conf```](https://github.com/calocaro/proyecto_apache/blob/master/confApache/ports.conf) donde estableceremos los puertos que escucha, en nuestro caso especificamos en la línea 5-6 los puertos 80 y 8000 indicando : **Listen 80** y **Listen 8000**.

```xml
# If you just change the port or add more ports here, you will likely also
# have to change the VirtualHost statement in
# /etc/apache2/sites-enabled/000-default.conf

Listen 80
Listen 8000

<IfModule ssl_module>
	Listen 443
</IfModule>

<IfModule mod_gnutls.c>
	Listen 443
</IfModule>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet

```

Seguidamente accedemos al apartado de [```sites-available```](https://github.com/calocaro/proyectoApache/tree/master/confApache/sites-available) donde copiaremos el fichero [```000-default.conf```](https://github.com/calocaro/proyecto_apache/blob/master/confApache/sites-available/000-default.conf) y haremos su réplica en [```002-default.conf```](https://github.com/calocaro/proyecto_apache/blob/master/confApache/sites-available/002-default.conf)

En cada uno establecemos el **Puerto** en la línea 1 y el **DocumentRoot** en la línea 12.

*Sitio1 000-default.conf*

```xml

<VirtualHost *:80>
	# The ServerName directive sets the request scheme, hostname and port that
	# the server uses to identify itself. This is used when creating
	# redirection URLs. In the context of virtual hosts, the ServerName
	# specifies what hostname must appear in the request's Host: header to
	# match this virtual host. For the default virtual host (this file) this
	# value is not decisive as it is used as a last resort host regardless.
	# However, you must set it for any further virtual host explicitly.
	#ServerName www.example.com

	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/html/sitio1

	# Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
	# error, crit, alert, emerg.
	# It is also possible to configure the loglevel for particular
	# modules, e.g.
	#LogLevel info ssl:warn

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

	# For most configuration files from conf-available/, which are
	# enabled or disabled at a global level, it is possible to
	# include a line for only one particular virtual host. For example the
	# following line enables the CGI configuration for this host only
	# after it has been globally disabled with "a2disconf".
	#Include conf-available/serve-cgi-bin.conf
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet


```

*Sitio2 002-default.conf*

```xml

<VirtualHost *:8000>
	# The ServerName directive sets the request scheme, hostname and port that
	# the server uses to identify itself. This is used when creating
	# redirection URLs. In the context of virtual hosts, the ServerName
	# specifies what hostname must appear in the request's Host: header to
	# match this virtual host. For the default virtual host (this file) this
	# value is not decisive as it is used as a last resort host regardless.
	# However, you must set it for any further virtual host explicitly.
	#ServerName www.example.com

	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/html/sitio2

	# Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
	# error, crit, alert, emerg.
	# It is also possible to configure the loglevel for particular
	# modules, e.g.
	#LogLevel info ssl:warn

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

	# For most configuration files from conf-available/, which are
	# enabled or disabled at a global level, it is possible to
	# include a line for only one particular virtual host. For example the
	# following line enables the CGI configuration for this host only
	# after it has been globally disabled with "a2disconf".
	#Include conf-available/serve-cgi-bin.conf
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet

```

Replicaremos los pases en el aspecto en el que en [```sites-enabled```](https://github.com/calocaro/proyecto_apache/tree/master/confApache/sites-enabled) donde replicaremos el fichero [```000-default.conf```](https://github.com/calocaro/proyecto_apache/blob/master/confApache/sites-enabled/000-default.conf) y haremos su réplica en [```002-default.conf```](https://github.com/calocaro/proyecto_apache/blob/master/confApache/sites-enabled/002-default.conf) y modicaremos los mismos parámetros que en [```sites-available```](https://github.com/calocaro/proyecto_apache/tree/master/confApache/sites-available).

Establecemos un ```index.php``` cualquiera en cada uno de nuestros sitios, una vez hagamos *docker-compose up* (Habiendo eliminado previamente *docker-compose down -v* los volúmenes) estos se visualizarán, si vamos a nuestro buscador podremos comprobar que el **sitio1** cargará con ```localhost:80```(por defecto es el :80 si introducimos localhost) y el **sitio2** en ```localhost:8000```.

*Comprobación*

_![GIF2](https://github.com/calocaro/proyecto_apache/blob/master/pictures/comprobacionSitiosPuertos.gif)_

# Integración servidor DNS

Primeramente para levantar el contenedor con un servidor de DNS y personalizarlo para nuestro caso particular, utilizaremos nuestro anterior proyecto [```Servidor DNS en Docker```](https://github.com/calocaro/Servidor-DNS-Docker).

Por lo que antes de nada deberemos mover todas las carpetas y archivos de dicho proyecto a [```proyectoApache```](https://github.com/calocaro/proyecto_apache), es decir, la carpeta [```conf```](https://github.com/calocaro/proyecto_apache/tree/master/conf) y la carpeta [```zonas```](https://github.com/calocaro/proyecto_apache/tree/master/zonas).

Antes de centrarnos en modificar los ficheros de configuración de fichero apache vamos a implementarlo en nuestro [```docker-compose.yml```](https://github.com/calocaro/proyecto_apache/blob/master/docker-compose.yml).

Para ello deberemos crear una nueva red y asignar a cada contenedor una IP diferente.

* **Contenedor bind9 con los sitios (Servidor DNS)**

*Añadimos el apartado ```networks``` y asignamos la IP 10.1.0.254 creada previamente denominada ```bind9_subnet```*.

```
 bind9:
    container_name: asir1_bind9
    image: internetsystemsconsortium/bind9:9.16
    ports:
      - 5300:53/udp
      - 5300:53/tcp
    networks:
      bind9_subnet:
        ipv4_address: 10.1.0.254
    volumes:
      - ./conf:/etc/bind
      - ./zonas:/var/lib/bind

```

* **Contenedor servidor Apache**

*En este caso añadimos nuevamente como en el caso del servidor DNS la misma network pero en este caso con la IP ```10.1.0.253```*

```
services:
  apache:
    container_name: apachePHP
    image: php:7.2-apache
    networks:
      bind9_subnet:
        ipv4_address: 10.1.0.253
    ports:
      - '80:80'
      - '8000:8000'
  
    volumes:
      - ./html:/var/www/html
      - ./confApache:/etc/apache2

```


[README.md](README.md) de Carolina Uruburo Zapata para el repositorio [Proyecto Apache](https://github.com/calocaro/proyecto_apache)
