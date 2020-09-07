## Trabajo Práctico 3 - Arquitectura de Sistemas Distribuidos

### 1- Objetivos de Aprendizaje
 - Familiarizarse con conceptos de Sistemas Distribuidos
 - Utilización avanzada de Docker.

### 2- Unidad temática que incluye este trabajo práctico
Este trabajo práctico corresponde a la unidad Nº: 2 (Libro Ingeniería de Software: Unidad 18)

### 3- Consignas a desarrollar en el trabajo práctico:

A continuación, se presentarán algunos conceptos avanzandos de Docker para poder configurar y utilizar sistemas complejos compuestos por varios servicios. Luego analizaremos estos sistemas para identificar sus partes y funcionamiento de los mismos.

#### Docker Network

Una de las razones por las que los contenedores y servicios de Docker son tan poderosos es que puede conectarlos entre sí o conectarlos a cargas de trabajo que no sean de Docker. Los contenedores y servicios de Docker ni siquiera necesitan saber que están implementados en Docker, o si sus pares también son cargas de trabajo de Docker o no. Ya sea que sus hosts Docker ejecuten Linux, Windows o una combinación de los dos, puede usar Docker para administrarlos de una manera independiente de la plataforma.

#### Bridge Network Driver

Para crear la red Docker utiliza distintos drivers, el más simple es el `bridge` driver, éste crea una red privada interna al host para que los contenedores de esta red puedan comunicarse. El acceso externo se otorga exponiendo los puertos a los contenedores. Docker protege la red mediante la administración de reglas que bloquean la conectividad entre diferentes redes Docker.

Detrás de escena, Docker Engine crea los puentes de Linux, las interfaces internas, las reglas de iptables y las rutas de host necesarios para hacer posible esta conectividad.

![alt text][imagen]

[imagen]: docker-bridge.png

(Imagen: https://www.docker.com/blog/understanding-docker-networking-drivers-use-cases/ )


#### Que es docker compose?

Si nuestro sistem distribuido está compuesto de varios componentes corriendo con Docker, al momento de compilar, ejecutar y conectar los contenedores desde Dockerfiles separados definitivamente requiere mucho tiempo. Entonces, como solución a este problema, Docker Compose nos permite usar un archivo YAML para definir aplicaciones de múltiples contenedores. Es posible configurar tantos contenedores como queramos, cómo se deben construir y conectar, y dónde se deben almacenar los datos. Podemos ejecutar un solo comando para compilar, ejecutar y configurar todos los contenedores cuando el archivo YAML esté completo.


## 4- Desarrollo:


#### 1- Sistema distribuido simple 
  - Ejecutar el siguiente comando para crear una red en docker
  ```bash
  docker network create -d bridge mybridge
  ```
  - Instanciar una base de datos Redis conectada a esa Red.
  ```bash
   docker run -d --net mybridge --name db redis:alpine
   ```
  - Levantar una aplicacion web, que utilice esta base de datos
  ```bash
    docker run -d --net mybridge -e REDIS_HOST=db -e REDIS_PORT=6379 -p 5000:5000 --name web alexisfr/flask-app:latest
  ```
  - Abrir un navegador y acceder a la URL: http://localhost:5000/
  - Verificar el estado de los contenedores y redes en Docker, describir:
    - ¿Cuáles puertos están abiertos?

        Los puertos abiertos son: 5000 para la app, y el 6379 para redis.

    - Mostrar detalles de la red `mybridge` con Docker.

```json
[
    {
        "Name": "mybridge",
        "Id": "19926540c8aea3271e89cc6ddc9f5182dafd5eadb0d78f512d5110a6cb238b06",
        "Created": "2020-09-07T16:38:51.0098879Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.20.0.0/16",
                    "Gateway": "172.20.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "3f4a21367cbddf96a4896fe479f3679348dce2585427e3b11e535b4a7748190a": {
                "Name": "web",
                "EndpointID": "de860eae1a7493138ec915f7752e6518e662b7bb51d85488e861ff1d4e41c821",
                "MacAddress": "02:42:ac:14:00:03",
                "IPv4Address": "172.20.0.3/16",
                "IPv6Address": ""
            },
            "c79df7fc6e2c6690dbf356827e121cd4c635f67b379887f4c6a68097f054ae53": {
                "Name": "db",
                "EndpointID": "b4d7314bf46403bdb3d5b392ea8608d8fa6930295c9bcef72f5d6cf7f786cbb2",
                "MacAddress": "02:42:ac:14:00:02",
                "IPv4Address": "172.20.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```

    - ¿Qué comandos utilizó?

         AR0FVFYQ1BHHV2J:~ lmurature$ docker network inspect mybridge

#### 2- Análisis del sistema 
  - Siendo el código de la aplicación web el siguiente:
```python
import os

from flask import Flask
from redis import Redis


app = Flask(__name__)
redis = Redis(host=os.environ['REDIS_HOST'], port=os.environ['REDIS_PORT'])
bind_port = int(os.environ['BIND_PORT'])


@app.route('/')
def hello():
    redis.incr('hits')
    total_hits = redis.get('hits').decode()
    return f'Hello from Redis! I have been seen {total_hits} times.'


if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True, port=bind_port)
```
  - Explicar cómo funciona el sistema

        El sistema cada vez que tiene un GET a la ruta '/' suma un hit a la base de redis y devuelve la info en texto plano.

  - ¿Para qué se sirven y porque están los parámetros `-e` en el segundo Docker run del ejercicio 1?

        Los parametros -e sirven para asignarle variables de entorno al container para no tener que hardcodear credenciales o datos sensibles.

  - ¿Qué pasa si ejecuta `docker rm -f web` y vuelve a correr ` docker run -d --net mybridge -e REDIS_HOST=db -e REDIS_PORT=6379 -p 5000:5000 --name web alexisfr/flask-app:latest` ?

        Se ejecuta la app de vuelta pero no se descarga la imagen.

  - ¿Qué occure en la página web cuando borro el contenedor de Redis con `docker rm -f db`?

        Excepción por error de conexión con Redis.

  - Y si lo levanto nuevamente con `docker run -d --net mybridge --name db redis:alpine` ?

        Vuelve a la normalidad pero perdio los datos que tenia guardados anteriormente.

  - ¿Qué considera usted que haría falta para no perder la cuenta de las visitas?

        Si, totalmente.

  - Para eliminar los elementos creados corremos:
  ```bash
  docker rm -f db
  docker rm -f web
  docker network rm mybridge
  ```
  
#### 3- Utilizando docker compose 
  - Normalmente viene como parte de la solucion cuando se instaló Docker
  - De ser necesario instalarlo hay que ejecutar:
  ```bash
  sudo pip install docker-compose
  ```
  - Crear el siguente archivo `docker-compose.yaml` en un directorio de trabajo:
  ```yaml
version: '3.6'
services:
  app:
    image: alexisfr/flask-app:latest
    depends_on:
      - db
    environment:
      - REDIS_HOST=db
      - REDIS_PORT=6379
    ports:
      - "5000:5000"
  db:
    image: redis:alpine
    volumes:
      - redis_data:/data
volumes:
  redis_data: 
  ```
  - Ejecutar `docker-compose up -d`
  - Acceder a la url http://localhost:5000/
  - Ejecutar `docker ps`, `docker network ls` y `docker volume ls`
  - ¿Qué hizo **Docker Compose** por nosotros? Explicar con detalle.

    Docker compose especifico que servicios necesitamos para levantar nuestra aplicacion distribuida
    Con el archivo especificamos que 'app' va a utilizar la imagen alexisfr/flask-app:latest,
    que depende de 'db', se le setea sus variables de entorno necesarias, y mapea los puertos con la
    maquina local. Para 'db' se especifica la imagen que va a correr y a que volumes hace referencia.

  - Desde el directorio donde se encuentra el archivo `docker-compose.yaml` ejecutar:
  ```bash
  docker-compose down
  ```
 
#### 4- Aumentando la complejidad, análisis de otro sistema distribuido.
Este es un sistema compuesto por:

- Una aplicación web de Python que te permite votar entre dos opciones
- Una cola de Redis que recolecta nuevos votos
- Un trabajador .NET o Java que consume votos y los almacena en...
- Una base de datos de Postgres respaldada por un volumen de Docker
- Una aplicación web Node.js que muestra los resultados de la votación en tiempo real.

Pasos:
- Clonar el repositorio https://github.com/dockersamples/example-voting-app
- Abrir una línea de comandos y ejecutar
```bash
cd example-voting-app
docker-compose -f docker-compose-javaworker.yml up -d
```
- Una vez terminado acceder a http://localhost:5000/ y http://localhost:5001
- Emitir un voto y ver el resultado en tiempo real.
- Para emitir más votos, abrir varios navegadores diferentes para poder hacerlo
- Explicar como está configurado el sistema, puertos, volumenes componenetes involucrados, utilizar el Docker compose como guía.

#### 5- Análisis detallado
- Exponer más puertos urtos para ver la configuración de Redis, y las tablas de PostgreSQL con alguna IDE como dbeaver.
- Revisar el código de la aplicación Python `example-voting-app\vote\app.py` para ver como envía votos a Redis.
- Revisar el código del worker `example-voting-app\worker\src\main\java\worker\Worker.java` para entender como procesa los datos.
- Revisar el código de la aplicacion que muestra los resultados `example-voting-app\result\server.js` para entender como muestra los valores.
- Escribir un documento de arquitectura sencillo, pero con un nivel de detalle moderado, que incluya algunos diagramas de bloques, de sequencia, etc y descripciones de los distintos componentes involucrados es este sistema y como interactuan entre sí.

### Presentación del trabajo práctico 3

La presentación de este práctico forma parte del trabajo integrador, especialemente el último punto con el analisis del sistema, todos los documentos e imagenes pueden ser subidos a una carpeta trabajo-practico-03 con las salidas de los comandos utilizados, explicaciones y respuestas a las preguntas.