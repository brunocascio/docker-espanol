# Docker
---

## Conceptos importantes:
  - Docker (daemon)
  - Docker-machine (client)
  - Docker Images
  - Docker Containers

## Introducción

 . . . . TODO . . . .

## Instalación

  . . . . TODO . . . .

## Empezando

### `ps`

Muestra los contenedores en ejecución (los que están corriendo en 2do plano).

```
  $ docker ps
```

Si lo ejecutamos luego del comando anterior, no mostrará nada porque lo anterior era solo un `build, run, die`.

Para ver el historial de ejecución y los contnedores actualmente creados (corriendo o no), ejecutamos:

```
  $ docker ps -a
```

Esto nos mostrará todos los contendores que se encuentran creados hasta el momento. Podes borrarlos si quisieramos `docker rm <container-id>`

### `images`

Muestra las imagenes locales disponibles.

```
  $ docker images
```

Podemos eliminarlas con `docker rmi <image-name>`, siempre y cuando no tenga contenedores asociados (corriendo o no).

### `events`

Podemos ver en tiempo real, los eventos que docker lanza en nuestro servidor, solo basta con:

```
  $ docker events
```

### `run`

Corremos un contenedor con la *imagen* base `busybox`, que ejecuta el comando `echo hello world` dentro. Luego de esto el contenedor se detendrá, porque de esta forma funciona como un "job".

```
  $ docker run busybox echo hello world
```

### `run` avanzado

Podemos pasar muchas opciones al comando run, las cuales podemos ver con:

```
  $ docker run --help
```
#### `run` interactivo

Probemos de ejecutar y usar una terminal en el contendor:

```
  $ docker run -t -i ubuntu:14.04 /bin/bash
```

  * `-t`: Aloca una tty
  * `-i`: Nos comunicamos con el contenedor de modo interactivo.

**NOTA:** Al salir del modo interactivo el contendor se detendrá.

#### `run` Detached Mode

Problema: Ya sabemos como correr un contenedor de manera interactiva, pero el problema es que el mismo al terminar de ejecutar la tarea, finaliza. Si se quieren hacer contenedores que corran servicios (por ejemplo un servidor web) el comando es el siguiente:

```
$ docker run -d -p 1234:1234 python:2.7 python -m SimpleHTTPServer 1234
```

Esto ejecuta un servidor python (SimpleHTTPServer module), en el puerto `1234`. El argumento `-p 1234:1234` le indica a docker que tiene que hacer un **port forwarding** del conetedor hacia el puerto `1234` de la maquina host.

Ahora podemos abrir un browser en la dirección `http://localhost:1234`.

**Algo más**

La opción `-d` hace que el contenedor corra en segundo plano. Esto nos permite ejecutar comandos sobre el mismo en cualquier momento mientras esté en ejecución. Por ejemplo:

`$ docker exec -ti <container-id> /bin/bash`

Aquí simplemente se abre una `tty` en modo `interativo`. Podrían hacerse otras cosas como cambiar el *working directory*, setear *variables de entorno*, etc. La lista completa puede verse de [acá](https://docs.docker.com/reference/run/)

## Ciclo de vida de un contenedor

Hasta ahora vimos como ejecutar un contendor tanto en foreground como en background (detached). Ahora veremos como manejar el ciclo completo de vida de un contenedor.
Docker provee de comandos como `create` , `start`, `stop`, `kill` , y `rm`. En todos ellos podría pasarse el argumento `-h` para ver las opciones disponibles.
Ejemplo: `docker create -h`

Más arriba vimos como correr un contendor en segundo plano (detached). Ahora veremos en el mismo ejemplo, pero con el comando `create`. La única diferencia que esta vez no especificaremos la opción `-d`. Una vez preparado, necesitaremos lanzar el contendor con `docker start`.

Ejemplo:

```
$ docker create -P --expose=8001 python:2.7 python -m SimpleHTTPServer 8001
  a842945e2414132011ae704b0c4a4184acc4016d199dfd4e7181c9b89092de13
$ docker ps -a
  CONTAINER ID IMAGE      COMMAND              CREATED       ... NAMES
  a842945e2414 python:2.7 "python -m SimpleHTT 8 seconds ago ... fervent_hodgkin
$ docker start a842945e2414
  a842945e2414
$ docker ps
  CONTAINER ID IMAGE      COMMAND              ... NAMES
  a842945e2414 python:2.7 "python -m SimpleHTT ... fervent_hodgkin
```

Siguiendo el ejemplo, para detener el contenedor se puede ejecutar cualquiera de los siguientes comandos:

  * `$ docker kill a842945e2414` (envía SIGKILL)
  * `$ docker stop a842945e2414` (envía SIGTERM).

Así mismo, pueden reiniciarse (Hace un `docker stop a842945e2414` y luego un `docker start a842945e2414`):

`$ docker restart a842945e2414`

ó destruirse:

`$ docker rm a842945e2414`

## Crear una imagen Docker con un Dockerfile

**Problema:**

Ya entendemos como se descargan las imágenes del *Docker Registry*. ¿Que pasa si ahora quisiéramos armar nuestras propias imagenes? (Para compartir, obvio 😜)

**Solución:**

Usando un [Dockerfile](https://docs.docker.com/engine/reference/builder/). Un `Dockerfile` es un archivo de texto, que describe los pasos (secuenciales) a seguir para preparar una imagen Docker. Esto incluye instalación de paquetes, creación de directorios, definición de variables de entorno, ETC.
Toda imagen que creemos, parte de una *base image*. Como en otro de los ejemplos, usábamos la imagen [busybox](https://busybox.net/about.html) la cual combina utilidades UNIX en un único y simple ejecutable.

**Comenzando:**

Crearemos nuestra propia imagen con la imagen base *busybox* y setearemos sólo una variable de entorno para mostrar el funcionamiento.

Crea el directorio pepe y se posiciona dentro de él:

`$ mkdir pepe && cd $_`

Creamos el Dockerfile:

`$ touch Dockerfile`

Escribimos las siguietes líneas dentro del `Dockerfile`:

```
FROM busybox

ENV foo=bar
```
Hecho esto, haremos un `build` de la imagen con el nombre `my-busybox`:

`$ docker build -t my-busybox .` (Chequear el . al final)

Si todo salió bien al hacer `docker images`, deberíamos encontrar nuestra imagen. ¡WALÁ!

## Ejemplo Real: Wordpress Dockerizado.

*Es un setup básico, no lo usaría en producción :)*

Para esto usaremos MySql y HTTPD (apache o nginx).

**Problema:**

Como Docker ejecuta procesos en *foreground*, necesitamos encontrar la forma de ejecutar varios de estos simultaneamente. La directiva `CMD` que veremos más adelante, sólo ejecutará una instrucción. Es decir, si tenemos varios `CMD` dentro de un *Dockerfile*, ejecutará sólo el último.

**Solución:**

Usando [Supervisor](http://supervisord.org/index.html) para monitorear y ejecutar MySql y HTTPD. Supervisor se encarga de controlar varios procesos y se ejecuta como cualquier otro programa.

Veremos diferentes formas de hacer esto. En principio crearemos todo dentro de un único contenedor, pero luego explotaremos al máximo los principios y características de Docker para hacerlo, por ejemplo separar servicios en diferentes contenedores y *linkearlos*.

### Usando Supervisor y en un único contenedor

Creamos el Dockerfile, con este contenido:

```
  # Imagen Base
  FROM ubuntu:14.04

  # Instalamos dependencias
    # apache2: Servidor Web
    # php5: Lenguaje de programacion PHP
    # php5-mysql: Driver de MySql para PHP
    # supervisor: Lanzadaror y Monitor de procesos
    # wget: Utilidad para obtener archivos via HTTP
  RUN apt-get update && apt-get -y install \
    apache2 \
    php5 \
    php5-mysql \
    supervisor \
    wget

  # mysql-server se instala con internvención del usuario,
  # pero como no es modo interactivo lo que hacemos es setearle las variables
  # con un valor.
  # Para simplificar hemos usado como usuario y contraseña de mysql 'root'
  RUN echo 'mysql-server mysql-server/root_password password root' | \
    debconf-set-selections && \
    echo 'mysql-server mysql-server/root_password_again password root' | \
    debconf-set-selections

  # Procesdemos ahora si, a instalar mysql-server
  RUN apt-get install -qqy mysql-server

  # Preparamos Wordpress
    # Obtenemos la última versión de wordpress
    # Descomprimimos
    # Copiamos el contenido dentro del root del servidor
    # Removemos el viejo index.html (mensaje de bienvenida de apache)
  RUN wget http://wordpress.org/latest.tar.gz && \
    tar xzvf latest.tar.gz && \
    cp -R ./wordpress/* /var/www/html && \
    rm /var/www/html/index.html

  # De esto se encargaría supervisor, pero como necesitamos crear la base de datos
  # ejecutamos a mysql en background y creamos la base de datos llamada wordpress
  RUN (/usr/bin/mysqld_safe &); sleep 5; mysqladmin -u root -proot create wordpress

  # Reemplazamos el archivo wp-config.php (más abajo lo creamos) a la carpeta de wordpress
  # Este archivo contiene la configuración de nuestro sitio
  COPY wp-config.php /var/www/html/wp-config.php

  # Copiamos el archivo de configuración de supervisor (más abajo lo creamos)
  COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

  # Le decimos al contenedor que tiene que hacer accesible al puerto 80 (en el que corre HTTPD)
  # para así nosotros poder acceder al mismo desde fuera
  EXPOSE 80

  # Lanzamos Supervisor como proceso Foreground de Docker
  # Este se encargará de lanzar simultaneamente los demás :D
  CMD ["/usr/bin/supervisord"]
```

Creamos el archivo `supervisor.conf` con este contenido:

```
[supervisord]
nodaemon=true

[program:mysqld]
command=/usr/bin/mysqld_safe
autostart=true
autorestart=true
user=root

[program:httpd]
command=/bin/bash -c "rm -rf /run/httpd/* && /usr/sbin/apachectl -D FOREGROUND"
```

Creamos el archivo `wp-config.php` con este contenido:

```php
  <?php
  /**
   * The base configurations of the WordPress.
   *
   * This file has the following configurations: MySQL settings, Table Prefix,
   * Secret Keys, and ABSPATH. You can find more information by visiting
   * {@link http://codex.wordpress.org/Editing_wp-config.php Editing wp-config.php}
   * Codex page. You can get the MySQL settings from your web host.
   *
   * This file is used by the wp-config.php creation script during the
   * installation. You don't have to use the web site, you can just copy this file
   * to "wp-config.php" and fill in the values.
   *
   * @package WordPress
   */

  // ** MySQL settings - You can get this info from your web host ** //
  /** The name of the database for WordPress */
  define('DB_NAME', 'wordpress');

  /** MySQL database username */
  define('DB_USER', 'root');

  /** MySQL database password */
  define('DB_PASSWORD', 'root');

  /** MySQL hostname */
  define('DB_HOST', 'localhost');

  /** Database Charset to use in creating database tables. */
  define('DB_CHARSET', 'utf8');

  /** The Database Collate type. Don't change this if in doubt. */
  define('DB_COLLATE', '');

  /**#@+
   * Authentication Unique Keys and Salts.
   *
   * Change these to different unique phrases!
   * You can generate these using the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}
   * You can change these at any point in time to invalidate all existing cookies. This will force all users to have to log in again.
   *
   * @since 2.6.0
   */
  define('AUTH_KEY',         'put your unique phrase here');
  define('SECURE_AUTH_KEY',  'put your unique phrase here');
  define('LOGGED_IN_KEY',    'put your unique phrase here');
  define('NONCE_KEY',        'put your unique phrase here');
  define('AUTH_SALT',        'put your unique phrase here');
  define('SECURE_AUTH_SALT', 'put your unique phrase here');
  define('LOGGED_IN_SALT',   'put your unique phrase here');
  define('NONCE_SALT',       'put your unique phrase here');

  /**#@-*/

  /**
   * WordPress Database Table prefix.
   *
   * You can have multiple installations in one database if you give each a unique
   * prefix. Only numbers, letters, and underscores please!
   */
  $table_prefix  = 'wp_';

  /**
   * For developers: WordPress debugging mode.
   *
   * Change this to true to enable the display of notices during development.
   * It is strongly recommended that plugin and theme developers use WP_DEBUG
   * in their development environments.
   */
  define('WP_DEBUG', false);

  /* That's all, stop editing! Happy blogging. */

  /** Absolute path to the WordPress directory. */
  if ( !defined('ABSPATH') )
  	define('ABSPATH', dirname(__FILE__) . '/');

  /** Sets up WordPress vars and included files. */
  require_once(ABSPATH . 'wp-settings.php');
```

Ahora sólo queda realizar el build de nuestra imagen y luego ejecutar un contenedor :)

```
$ docker build -t wordpress .
$ docker run -d -p 80:80 wordpress
```

Una vez funcionando, ingresando en `http://<IP_OF_DOCKER_HOST>` deberíamos visualizar la página de instalación de wordpress.

**Nota:**

Usar Supervisor para ejecutar varios servicios dentro del mismo contenedor, podría trabajar perfectamente, pero es mejor usar múltiples contenedores. Estos proveen del aislamiento (isolation) ente otras bondades de Docker, y nos ayuda además a crear una aplicación basada en [microservicios](http://bit.ly/building-microservices). Por último, también esto nos ayuda a escalar y a recuperarnos de posibles fallas.

## Corriendo Wordpress usando 2 contenedores linkeados.

**Problema:**

Hasta ahora ejecutamos una instancia de wordpress con su servidor y su base de datos, en un mismo contenedor. El problema es queno explotamos al máximo a Docker, y no mantenemos tampoco el concepto de *Separation of concerns*. Necesitamos desacoplar el contendor lo mas fino posible.

**Solución:**

Usar 2 contenedors. Uno para Wordpress y otro para MySql. Luego se interconectaran mediante la opción de docker `--link`.

**Manos a la obra:**

Para este ejemplo usaremos las imagenes docker oficiales de wordpress y mysql.

```
$ docker pull wordpress:latest
$ docker pull mysql:latest
```

**Ejecutamos un contenedor MySql**

```
$ docker run --name mysqlwp -e MYSQL_ROOT_PASSWORD=wordpressdocker \
                          -e MYSQL_DATABASE=wordpress \
                          -e MYSQL_USER=wordpress \
                          -e MYSQL_PASSWORD=wordpresspwd \
                          -v /db/mysql:/var/lib/mysql \
                          -d mysql
```

NOTA: Aquí hay nuevas opciones:

  * `-e` es para setear variables de entorno. Esas variables están definidas dentro del Dockerfile de MySql, por lo que nosotros le damos valor, para que el contendor a ejecutar, use esos datos.
  * `-v` es para montar un volumen entre el host y el contenedor. En este caso en el host se populara el volumen `/db/mysql/` con la info de `/var/lib/mysql`.
    * Los volúmenes tienen diferentes usos:
      * Se crean cuando se inicializa el contenedor
      * Compartir información entre diferentes contenedores
      * Mantener la info luego de haber borrado el contendor
      * Cambios en los volúmenes son directamente aplicados (no hay que hacer nada con adicional con el contendor para actualizar)
      * Los cambios de un volumen no se incluirán en la actualización de la imagen  

**Ejecutamos y linkeamos a wordpress**

```
$ docker run --name wordpress --link mysqlwp:mysql -p 80:80 \
                              -e WORDPRESS_DB_NAME=wordpress \
                              -e WORDPRESS_DB_USER=wordpress \
                              -e WORDPRESS_DB_PASSWORD=wordpresspwd \
                              -d wordpress
```

NOTA: La imagen de wordpress, expone el puerto 80 y lo que hacemos es mapearlo con el 80 del nuestro Host. Como en la imagen de MySql, en wordpress tambien contamos con algunas variables de entorno, éstas para la configuración del mismo. Básicamente seteamos las credenciales de la base de datos anteriormente creada, para que wordpress use las mismas.

## Haciendo backups de la base de datos de un contenedor

**Problema:**

Tenemos un contenedor de mysql ejecutando, pero necesitamos hacer un backup de la base de datos que se ejecuta dentro del contenedor

**Solucion:**

Usar el comando `docker exec` para ejecutar en el contenedor MySql el comando `mysqldump`

Chequeamos en nuestro host que existe la carpeta `/db/mysql`

`$ ls /db/mysql`

Ahora, para hacer un backup de la base de datos de ese contenedor ejecutamos:

`docker exec mysqlwp mysqldump --all-databases --password=wordpressdocker > wordpress.backup`

Ahora ejecutamos `$ ls` y veremos el archivo `wordpress.backup` :)
