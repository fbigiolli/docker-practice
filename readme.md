# Practica Docker

Este repositorio contiene un proyecto que armé para dar los primeros pasos con Docker. La idea es dejar el paso a paso lo mejor explicado posible en este readme.

## Sobre la app

Es la app del [tutorial de Django REST](https://www.django-rest-framework.org/tutorial/1-serialization/).

## Instalacion y setup

En primer lugar, hay que descargar [Docker desktop](https://docs.docker.com/get-docker/). Una vez instalado, para trabajar desde la terminal **hay que abrir la app para que levante el docker engine.**

## Creando el container

Hay que crear un archivo llamado **Dockerfile** en el directorio del proyecto. Para este proyecto uso la [imagen python sobre alpine](), se podrian usar otras imagenes. Incluso se puede usar por ejemplo una de Debian, pero esto implicaria tener que instalar todas las dependencias para el proyecto. La imagen basada en alpine tiene como ventaja que pesa muy poco.  

Se ve la imagen seleccionada en la primer linea del Dockerfile:

```
FROM python:3.13.0rc1-alpine3.20
```

Docker necesita conocer las dependencias. Para crear el archivo con las dependencias necesarias de nuestro proyecto, sobre la raiz hay que correr:

```
pip freeze > requirements.txt
```

>[!NOTE]
>Es importante activar el env para que no se agreguen todas las librerias instaladas en el sistema al requirements. Al activar el env solamente se agregan las relacionadas al proyecto.

Las proximas lineas del Dockerfile son:

Le indica a docker que /app es el directorio de trabajo dentro de la imagen
```
WORKDIR /app
```

Copia los requisitos en el directorio y con el comando RUN los instala

```
COPY requirements.txt /app/
RUN pip install --no-cache-dir -r requirements.txt
```
Copia el resto del codigo de la aplicacion

```
COPY . /app/
```

Indica que el container debe exponer el puerto 8000, que va ser por el que nos vamos a comunicar con la app

```
EXPOSE 8000
```

Ejecuta el comando de python que hace correr el server de Django en el puerto 8000

```
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

## Buildeando la imagen y probando

Una vez creado el Dockerfile, en el directorio del proyecto corremos el comando: 
```
docker build -t *nombre del container* .
```

La flag -t sirve para ponerle un tag al container, lo cual nos ayuda despues a identificarlo.

Ya creado el container, lo corremos con el comando: 
```
docker run -p 8000:8000 *nombre del container* 
```

La flag -p es una forma de mapear el puerto 8000 del container al puerto 8000 de nuestra red local. De esta forma, estamos exponiendo la app que corre en nuestro container en el puerto 8000 de nuestra red. Es importante que el mapeo sea correcto, en este caso se hace 8000:8000 porque le dijimos a la app que corra en el puerto 8000 del container, y la vamos a acceder en el puerto 8000 de nuestra red. 


## Docker Compose

Para practicar Docker Compose, la idea es armar dos containers que se van a comunicar entre si. En este caso, los containers van a ser dos: uno para la app, y otro para la DB

## Migración de SQLite a PostgreSQL

Por default, la app de Django viene configurada para correr la DB en SQLite. Antes de crear ambos containers, voy a migrar la DB de SQLite a PostgreSQL. PostgreSQL se puede instalar desde [esta pagina](https://www.postgresql.org/download/windows/).

En primer lugar, hay que instalar **psycopg2** (yo instale la version binary) que nos permite conectar django con postgre. Para eso, con env activado:

```
pip install psycopg2-binary
```

No voy a comentar toda la creacion de la db de postgre porque es bastante larga, paso directamente a la parte de docker compose con postgre instalado y la db creada.

## Usando Docker-Compose
En primer lugar tuve que cambiar la imagen, ya que la version alpine me daba problemas para hacer build con el contenedor de Django. En el dockerfile agregue las siguientes lineas (no estoy seguro si son estrictamente necesarias):

```
RUN apt-get update && \
    apt-get install -y \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*
```

Hice una breve modificacion en /tutorial/settings.py:

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.getenv('POSTGRES_DB', 'django-test'),
        'USER': os.getenv('POSTGRES_USER', 'postgres'),
        'PASSWORD': os.getenv('POSTGRES_PASSWORD', '123'),
        'HOST': 'db', 
        'PORT': '5432',
    }
}
```

De esta forma, con os.getenv el proyecto obtiene el valor de las variables de entorno, y en caso que no pudiera encontrarlo, va a usar el valor por default que esta como segundo parametro en la funcion getenv. (Importante no olvidarse de hacer el **import os** correspondiente para poder usar la funcion)

## Creando docker-compose.yml

Para usar docker compose, hay que crear un archivo en la raiz del proyecto llamado **docker-compose.yml**.
En el hay que especificar los servicios para los cuales queremos que se creen containers. Tambien se especifican variables de entorno, volumenes, puertos y dependencias entre containers. Es necesario poner un user de la db que tenga acceso a la db que justamente queremos crear. En este caso, yo use el user postgres.

Una vez hecho esto, se corren los siguientes comandos, el primero para hacer el build de los containers, y el segundo para levantarlos:

```
docker-compose build
docker-compose up
```

Y nuevamente en localhost:8000 está funcionando la app.

## Migraciones

Si queremos ir mas alla del index, Django nos va a mostrar un error diciendo que no existen las tablas.

Al hacer el cambio de db, como estamos armando una db nueva en un container es necesario aplicar las migraciones de Django para que cree las tablas nuevamente. Para ello, con el container ya corriendo, en una terminal nueva hay que usar los siguientes comandos:

```
docker-compose exec web python manage.py makemigrations
docker-compose exec web python manage.py migrate
```

En este caso, hago exec **web** porque el servicio que le asigne a Django en el archivo docker-compose.yml es ese. Basicamente es decirle a docker que ese comando lo queremos usar sobre el container llamado web.

Finalmente, la app esta operativa otra vez, pero ahora dockerizada en dos containers: uno para la db y otro para la app :)