## Trabajo Práctico 2 - Introducción a Docker

### 1- Objetivos de Aprendizaje
 - Familiarizarse con la tecnología de contendores 
 - Ejercitar comandos básicos de Docker.

### 2- Unidad temática que incluye este trabajo práctico
Este trabajo práctico corresponde a la unidad Nº: 2 (Libro Ingeniería de Software: Unidad 18)

### 3- Consignas a desarrollar en el trabajo práctico:

A continuación, se presentarán algunos conceptos generales de la tecnología de contenedores a manera de introducción al tema desde el punto de vista práctico.

#### Que son los contenedores?

Los contenedores son paquetes de software. Ellos contienen la aplicación a ejecutar junto con las librerías, archivos de configuración, etc para que esta aplicación pueda ser ejecutada. Estos contenedores utilizan características del sistema operativo, por ejemplo, cgroups, namespaces y otros aislamientos de recursos (sistema de archivos, red, etc) para proveer un entorno aislado de ejecución de dicha aplicación.

Dado que ellos utilizan el kernel del sistema operativo en el que se ejecutan, no tienen el elevado consumo de recursos que por ejemplo tienen las máquinas virtuales, las cuales corren su propio sistema operativo.

#### Que es docker?

Docker es una herramienta que permite el despliegue de aplicaciones en contenedores. Además, provee una solución integrada tanto para la ejecución como para la creación de contenedores entre otras muchas funcionalidades.

#### Porque usar contenedores?

Los contenedores ofrecen un mecanismo de empaquetado lógico en el cual las aplicaciones pueden estar aisladas del entorno en el cual efectivamente se ejecutan. Este desacoplamiento permite a las aplicaciones en contenedores ser desplegadas de manera simple y consistente independientemente de si se trata de un Data Center privado, una Cloud publica, o una computadora de uso personal. Esto permite a los desarrolladores crear entornos predecibles que están aislados del resto de las aplicaciones y pueden ser ejecutados en cualquier lugar.

Por otro lado, ofrecen un control más fino de los recursos y son más eficientes al momento de la ejecución que una máquina virtual.

En los últimos años el uso de contenedores ha crecido exponencialmente y fue adoptado de forma masiva por prácticamente todas las compañías importantes de software.

#### Máquinas Virtuales vs Contenedores 

Los contenderos no fueron pensados como un remplazo de las máquinas virtuales. Cuando ambas tecnologías se utilizan en forma conjunta se obtienen los mejores resultados, por ejemplo, en los proveedores cloud como AWS, Google Cloud o Microsoft Azure.

![alt text][imagen]

[imagen]: vms-vs-containers.png

(Imagen: https://blog.docker.com/2016/04/containers-and-vms-together/ )


##### Analogía

![alt text][imagen3]

[imagen3]: vms-containers-analogy.png

(Imagen: https://github.com/SteveLasker/Presentations/tree/master/DockerCon2017 )

#### Conceptos Generales

- **Container Image**: Una imagen contiene el sistema operativo base, la aplicación y todas sus dependencias necesarias para un despliegue rápido del contenedor.
- **Container**: Es una instancia en ejecución de una imagen.
- **Container Registry**: Las imágenes de Docker son almacenadas en un Registry y pueden ser descargadas cuando se necesitan. Un registry pude ser público, por ejemplo, DockerHub o instalado en un entorno privado.
- **Docker Daemon**: el servicio en segundo plano que se ejecuta en el host que gestiona la construcción, ejecución y distribución de contenedores Docker. El daemon es el proceso que se ejecuta en el sistema operativo con el que los clientes hablan.
- **Docker Client**: la herramienta de línea de comandos que permite al usuario interactuar con el daemon. En términos más generales, también puede haber otras formas de clientes, como Kitematic, que proporciona una GUI a los usuarios. 
- **Dockerfile**: Son usados por los desarrolladores para automatizar la creación de imágenes de contenedores. Con un Dockerfile, el demonio de Docker puede automáticamente construir una imagen.

#### Layers en Docker

Las imágenes de docker están compuestas de varias capas (layers) de sistemas de archivos y agrupadas juntas. Estas son de solo lectura. Cuando se crea el contenedor, Docker monta un systema de archivos de lectura/escritura sobre estas capas el cual es utilizado por los procesos dentro del contenedor. Cuando el contenedor es borrado, esta capa es borrada con él, por lo tanto son necesarias otras soluciones para persistir datos en forma permanente.

![alt text][imagen2]

[imagen2]: docker-image.png

(Imagen: https://washraf.gitbooks.io/the-docker-ecosystem/content/Chapter%201/Section%203/union_file_system.html)

## 4- Desarrollo:

#### 1- Instalar Docker Community Edition 
  - Diferentes opciones para cada sistema operativo
  - https://docs.docker.com/
  - Ejecutar el siguiente comando para comprobar versiones de cliente y demonio.
```bash
docker version
```
```bash
Client: Docker Engine - Community
 Version:           19.03.5
 API version:       1.40
 Go version:        go1.12.12
 Git commit:        633a0ea
 Built:             Wed Nov 13 07:22:34 2019
 OS/Arch:           darwin/amd64
 Experimental:      false
```

#### 2- Explorar DockerHub
   - Registrase en docker hub: https://hub.docker.com/
   - Familiarizarse con el portal

#### 3- Obtener la imagen BusyBox
  - Ejecutar el siguiente comando, para bajar una imagen de DockerHub
  ```bash
  docker pull busybox
  ```

Output del comando:
```bash
Using default tag: latest
latest: Pulling from library/busybox
61c5ed1cbdf8: Pull complete
Digest: sha256:4f47c01fa91355af2865ac10fef5bf6ec9c7f42ad2321377c21e844427972977
Status: Downloaded newer image for busybox:latest
docker.io/library/busybox:latest
```

  - Verificar qué versión y tamaño tiene la imagen bajada, obtener una lista de imágenes locales:
```bash
docker images
```

```bash 
REPOSITORY                                 TAG                 IMAGE ID            CREATED             SIZE
busybox                                    latest              018c9d7b792b        13 days ago         1.22MB
volunteer_api                              latest              d2759c9a4791        3 weeks ago         986MB
fury-cbt-translation-api                   latest              79e7bce162cf        2 months ago        1.01GB
golang                                     1.13                7ec6e7161786        3 months ago        803MB
golang                                     1.14                2421885b04da        3 months ago        809MB
fury-cbt-items-api                         latest              b0a162ec4bdb        3 months ago        1.04GB
hub.furycloud.io/mercadolibre/golang       1.12.7              663b18b7b98d        7 months ago        1.04GB
hub.furycloud.io/mercadolibre/java-maven   jdk8                7e3638b720f1        8 months ago        1.01GB
hub.furycloud.io/mercadolibre/golang       <none>              84450b3aa14c        11 months ago       1.04GB
hub.furycloud.io/mercadolibre/nginx        latest              5f2b3098857b        12 months ago       743MB
adamdoupe/wackopicko                       latest              c7bcfaf7c503        2 years ago         608MB
```

#### 4- Ejecutando contenedores
  - Ejecutar un contenedor utilizando el comando **run** de docker:
```bash
docker run busybox
```

  - Explicar porque no se obtuvo ningún resultado

  Porque no se les mandó ningun comando a correr en el contenedor.

  - Especificamos algún comando a correr dentro del contendor, ejecutar por ejemplo:
```bash
docker run busybox echo "Hola Mundo"
```

  - Ver los contendores ejecutados utilizando el comando **ps**:
```bash
docker ps
```
  - Vemos que no existe nada en ejecución, correr entonces:
```bash
docker ps -a
```
  - Mostrar el resultado y explicar que se obtuvo como salida del comando anterior.

```bash
CONTAINER ID        IMAGE               COMMAND               CREATED             STATUS                      PORTS               NAMES
a5b76f0c46e3        busybox             "echo 'Hola Mundo'"   18 seconds ago      Exited (0) 17 seconds ago                       peaceful_burnell
```

Se obtiene esto debido a que con la flag -a aparecen todas las ejecuciones en los contenedores de docker.

#### 5- Ejecutando en modo interactivo

  - Ejecutar el siguiente comando
```bash
docker run -it busybox sh
```
  - Para cada uno de los siguientes comandos dentro de contenedor, mostrar los resultados:
```bash
ps
```bash
/ # ps
PID   USER     TIME  COMMAND
    1 root      0:00 sh
    6 root      0:00 ps
```
uptime
```bash
/ # uptime
 22:09:26 up  6:08,  0 users,  load average: 0.00, 0.02, 0.00
```
free
```bash
/ # free
              total        used        free      shared  buff/cache   available
Mem:        2047132      290416     1297524         792      459192     1613504
Swap:       1048572           0     1048572
```
ls -l /
```bash
/ # ls -l /
total 36
drwxr-xr-x    2 root     root         12288 Jul 27 19:19 bin
drwxr-xr-x    5 root     root           360 Aug 10 22:08 dev
drwxr-xr-x    1 root     root          4096 Aug 10 22:08 etc
drwxr-xr-x    2 nobody   nogroup       4096 Jul 27 19:19 home
dr-xr-xr-x  183 root     root             0 Aug 10 22:08 proc
drwx------    1 root     root          4096 Aug 10 22:09 root
dr-xr-xr-x   13 root     root             0 Aug 10 22:08 sys
drwxrwxrwt    2 root     root          4096 Jul 27 19:19 tmp
drwxr-xr-x    3 root     root          4096 Jul 27 19:19 usr
drwxr-xr-x    4 root     root          4096 Jul 27 19:19 var
```
```
  - Salimos del contendor con:
```bash
exit
```

#### 6- Borrando contendores terminados

  - Obtener la lista de contendores 
```bash
docker ps -a
```
  - Para borrar podemos utilizar el id o el nombre (autogenerado si no se especifica) de contendor que se desee, por ejemplo:
```bash
docker rm elated_lalande
```

```bash
AR0FVFYQ1BHHV2J:~ lmurature$ docker rm 24b5a5b34e23
24b5a5b34e23
```

  - Para borrar todos los contendores que no estén corriendo, ejecutar cualquiera de los siguientes comandos:
```bash
docker rm $(docker ps -a -q -f status=exited)
```
```bash
docker container prune
```

#### 7- Montando volúmenes

Hasta este punto los contenedores ejecutados no tenían contacto con el exterior, ellos corrían en su propio entorno hasta que terminaran su ejecución. Ahora veremos como montar un volumen dentro del contenedor para visualizar por ejemplo archivos del sistema huésped:

  - Ejecutar el siguiente comando, cambiar myusuario por el usuario que corresponda. En linux/Mac puede utilizarse /home/miusuario):
```bash
docker run -it -v C:\Users\misuario\Desktop:/var/escritorio busybox /bin/sh
```

```bash
AR0FVFYQ1BHHV2J:~ lmurature$ docker run -it -v /Users/lmurature:/var/escritorio busybox /bin/sh
```

  - Dentro del contenedor correr
```bash
ls -l /var/escritorio
touch /var/escritorio/hola.txt
```
  - Verificar que el Archivo se ha creado en el escritorio o en el directorio home según corresponda.

```bash
ls var/escritorio/
Applications            Music                   go
Desktop                 Pictures                google-auth-meli.txt
Documents               Postman                 hola.txt
Downloads               Public                  jhbuild
GroovyApps              PythonProjects          js
JavaProjects            api-rules               tmp
Library                 error-delete-items.csv  ucc_ing-soft-3
Movies                  firmware                voluntariado_local
```

#### 8- Publicando puertos

En el caso de aplicaciones web o base de datos donde se interactúa con estas aplicaciones a través de un puerto al cual hay que acceder, estos puertos están visibles solo dentro del contenedor. Si queremos acceder desde el exterior deberemos exponerlos.

  - Ejecutar la siguiente imagen, en este caso utilizamos la bandera -d (detach) para que nos devuelva el control de la consola:

```bash
docker run -d daviey/nyan-cat-web
```
  - Si ejecutamos un comando ps:
```bash
PS D:\> docker ps
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS               NAMES
87d1c5f44809        daviey/nyan-cat-web   "nginx -g 'daemon of…"   2 minutes ago       Up 2 minutes        80/tcp, 443/tcp     compassionate_raman
```
  - Vemos que el contendor expone 2 puertos el 80 y el 443, pero si intentamos en un navegador acceder a http://localhost no sucede nada.

  - Procedemos entonces a parar y remover este contenedor:
```bash
docker kill compassionate_raman
docker rm compassionate_raman
```
  - Vamos a volver a correrlo otra vez, pero publicando uno de los puertos solamente, el este caso el 80

```bash
docker run -d -p 80:80 daviey/nyan-cat-web
```
  - Accedamos nuevamente a http://localhost y expliquemos que sucede.

  Lo que sucede es que con -p hacemos un mapping de puertos de nuestra computadora a puertos del contenedor. Entonces si intentamos acceder al localhost al 80, nos redireccionará al 80 del container.

#### 9- Utilizando una base de datos
- Levantar una base de datos PostgreSQL

```bash
mkdir $HOME/.postgres

docker run --name my-postgres -e POSTGRES_PASSWORD=mysecretpassword -v $HOME/.postgres:/var/lib/postgresql/data -p 5432:5432 -d postgres:9.4
```
- Ejecutar sentencias utilizando esta instancia

```bash
docker exec -it my-postgres /bin/bash

psql -h localhost -U postgres

#Estos comandos se corren una vez conectados a la base

\l
create database test;
\connect test
create table tabla_a (mensaje varchar(50));
insert into tabla_a (mensaje) values('Hola mundo!');
select * from tabla_a;

\q

exit
```

- Conectarse a la base utilizando alguna IDE (Dbeaver - https://dbeaver.io/, eclipse, IntelliJ, etc...). Interactuar con los objectos objectos creados.

- Explicar que se logro con el comando `docker run` y `docker exec` ejecutados en este ejercicio.

Con docker run lo que hicimos fue empezar a correr el contenedor con la imagen adentro, para que la base este corriendo y poder acceder a ella. Con docker exec, logramos entrar dentro del contenedor de forma local para utilizar psql como si fuera nuestra maquina local.

#### 10- Presentación del trabajo práctico.

Subir un archivo md (puede ser en una carpeta) trabajo-practico-02 con las salidas de los comandos utilizados. Si es necesario incluir también capturas de pantalla.