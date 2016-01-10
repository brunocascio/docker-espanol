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
