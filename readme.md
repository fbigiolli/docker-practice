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

## Construyendo la imagen y probando

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