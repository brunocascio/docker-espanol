FROM ubuntu:14.04

# Actualizamos repositorios e instalamos dependencias.
RUN apt update
RUN apt install -y python python-pip
RUN apt-get clean all
RUN pip install flask

# Agregamos nuestra aplicación al Filesystem del contenedor.
ADD hello.py /tmp/hello.py

# Exponemos el puerto del contenedor
EXPOSE 5000

# Comando por default que se ejecuta cuando se corre el contenedor
CMD ["python","/tmp/hello.py"]
