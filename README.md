# Créditos

Esta aplicación shiny y la idea general se encuentran en el repositorio 

https://github.com/sellorm/shiny-app-docker.git


# Estructura del proyecto


La distribución de carpetas propuesta es la siguiente. Se puede adaptar a la que se considere en cada app, pero conviene entender primero qué es cada cosa.

```
.
./data-prep
./data-prep/raw-data.csv
./data-prep/data-prep-script.R
./Dockerfile
./shiny-app-docker.Rproj
./README.md
./.gitignore
./shiny-app
./shiny-app/app.R
./shiny-app/data-df.rds
```


Importantes:

**data-prep**, carpeta donde hay scrips de preprocesamiento de datos CSV. Ejemplifica como podemos tener otras cosas en nuestro proyecto y no necesariamente participan del despliegue en Docker.

**shiny-app**, la carpeta principal de la app

**Dockerfile**, el fichero que compone la imagen docker. Analizado más abajo.



## Dockerfile y layout


Como se ha dicho el Dockerfile contiene la receta para construir la imagen docker: la imagen base, las dependencias adicionales, el código propio y la forma de ejecutar. 

```
# La base de nuestra imagen docker
# get shiny serveR and a version of R from the rocker project
FROM rocker/shiny:4.0.5

# Dependencias
# Intenta añadir lo mínimo imprescindible
# Package Manager is a good resource to help discover system deps
RUN apt-get update && apt-get install -y \
    libcurl4-gnutls-dev \
    libssl-dev


# Instala los paquetes R necesarios
RUN R -e 'install.packages(c(\
              "shiny", \
              "shinydashboard", \
              "ggplot2" \
            ), \
            repos="https://packagemanager.rstudio.com/cran/__linux__/focal/2021-04-23"\
          )'


# Copiamos la aplicación dentro de la imagen
COPY ./shiny-app/* /srv/shiny-server/

# run app
CMD ["/usr/bin/shiny-server"]
```


Comentarios:

  - Las dependencias de R hay que ponerlas explícitamente. Cada proyecto puede tener las suyas.
  - **data-prep** no se requiere para correr la shiny, por lo que no se incluye en la imagen
  - La última linea del fichero lanza el proceso shiny  


# Arrancamos la aplicación

Para hacer una ejecución local tenemos que instalar docker en nuestro ordenador. Es un servicio que ya lo vamos a tener para cualquier otra experiencia posterior. Imprescindible.

Docker engine install: https://docs.docker.com/engine/


Una vez instalado podemos tener un panel gráfico en Windows o la linea de comandos tanto en linux como en Mac. El resto de documento lo basamos en linea de comandos.


## 1.- Creamos la imagen docker


Por ejemplo, vamos a crear la imagen con el nombre del alumno. Nos situamos en la base del proyecto, donde está el Dockerfile:

```
docker build -t alumno01-app .
```


No debe haber errores de creación de la imagen. Ojo con la sintaxis del Dockerfile si lo editamos finalmente.

## 2.- Arrancamos la aplicación

Tenemos que publicar el puerto de shiny (default 3838) hacia el exterior del contenedor docker para que sea accesible.


```
docker run --rm -p 3838:3838 alumno01-app
```

El comentario que cabe hace es que usando la imagen "alumno01-app" creamos una instancia docker que permitirá que usando el puerto 3838 externo se comunique con el puerto interno 3838. Hay más opciones, como mapear volúmenes externos, etiquetar la imagen, etc..


La ventana de terminal queda mostrando los logs.


## 3.- Ejecutamos la aplicación

Abrimos en un navegador la url http://localhost:3838

Los logs aparecen en la consola desde la que hemos arrancado la instancia.
Un contenedor docker no tiene persistencia, por lo que ni logs ni datos modificados se conservarán. 


# Otros comandos útiles


## Imágenes 


Como cada vez que ejecutemos docker build se puede crear una imagen docker nueva, se va a tender a llenar de imágenes el repositorio local. Algunos de estos comandos son útiles para gestionar las imágenes

```
$ docker image ls #listamos imágenes

REPOSITORY     TAG             IMAGE ID       CREATED        SIZE
my-shiny-app   latest          111d0409178e   46 hours ago   1.45GB
haproxy        latest          64ed6f31fba6   6 days ago     1.98GB
<none>         <none>          2412549e9be8   6 days ago     1.98GB
<none>         <none>          9fe97a29c6b8   6 days ago     1.98GB
```

El consejo es eliminar las imágenes que no nos sirven. 

```
$ docker image rm 9fe97a29c6b8  #borramos imagen
```

## Instancias 

También se quedan las instancias en nuestro almacenamiento y se deben eliminar si no son necesarias.

```
$ docker ps    # las instancias que están vivas

CONTAINER ID   IMAGE          COMMAND                  CREATED        STATUS        PORTS                    NAMES
7770676bd20c   my-shiny-app   "/usr/bin/shiny-serv…"   46 hours ago   Up 46 hours   0.0.0.0:3838->3838/tcp   xenodochial_mestorf
```

```

$ docker ps -a  # todas las instancias 
CONTAINER ID   IMAGE          COMMAND                  CREATED        STATUS                   PORTS                    NAMES
7770676bd20c   my-shiny-app   "/usr/bin/shiny-serv…"   46 hours ago   Up 46 hours              0.0.0.0:3838->3838/tcp   xenodochial_mestorf
4853da7a31be   haproxy        "/tini -- /nginx/sta…"   6 days ago     Up 6 days                0.0.0.0:8080->8080/tcp   keen_moser
```


```
$ docker rm -f 4853da7a31be  #borraría la instancia incluso parándola si es necesario
$ docker stop 4853da7a31be
$ docker start 4853da7a31be
$ docker exec -it 4853da7a31be bash  #abrimos una consola de shell en la instancia

```


