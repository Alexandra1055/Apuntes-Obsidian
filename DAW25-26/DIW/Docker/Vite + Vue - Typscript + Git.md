Primeros pasos:
VITE
https://vite.dev/guide/
DOCKER
https://hub.docker.com/
Creamos un Docker-compose.yml con la siguiente estructura:
```yaml
name: "examen-prueba"
services:
 vite:
  build:
   context: .
   dockerfile: ./vite/dockerfile
  tty: true
  stdin_open: true
  working_dir: /app
  volumes:
    - ./vite:/app
  ports:
    - "8082:5173"
```

Y la carpeta con el dockerfile:
Vite -> dockerfile
```dockerfile
FROM node:24
RUN apt update
```

Iniciamos el contenedor: `docker compose up -d --build`

Abrimos el contenedor desde docker, y entramos en la terminal: Exec
>[!note] para poder verlo como una terminal, usamos el comando `/bin/bash`

```bash
# Comando para ejecutar Vite
npm create vite@latest
```

Una vez dentro tendremos varias opciones, y tras elegir se crea la carpeta
>[!warning] si queremos que se cree en la carpeta que estamos trabajando añadimos " . " al crearlo:
>```bash 
>npm create vite@latest .
>```


##### SCSS
Si nos pide que este dentro del contenedor de Vite, solo tendríamos que añadir en el dockerfile de vite (después de haberlo creado)

```dockerfile
FROM node:24
RUN apt update
COPY ./vite/package.json .
RUN npm install -D sass-embedded
```

Si pide que tenga su imagen en el docker-compose
```yaml
  scss:
    build:
      context: .
      dockerfile: Dockerfile-scss
    tty: true
    stdin_open: true #tty + stdin = -it
    working_dir: /app #-w /app
    volumes:
      - ./scss:/app
```

Faltaria añadir un fichero dentro del src:  style.scss
```scss
$bg: lightcoral;
$text: #111;

body {
  background-color: $bg;
  color: $text;
}

h1 {
  text-decoration: underline;
}
```

y despues en la carpeta main.ts, tsx o .js lo que salga cambiamos el import:
```ts
import './style.scss'
```


#### CONTENEDOR CON GIT
En el docker compose:
```yaml
 demo6:
  build:
   context: .
   dockerfile: ./demo/dockerfile
  tty: true
  stdin_open: true
  working_dir: /app/webpack-demos/demo06 #directorio en el que trabajara
  volumes:
    - ./demo/demo-build:/build/demo # de la carpeta demo-build a la carpeta interna build/demo
  ports:
    - "8083:5173"
```

Todos los proyectos en docker tienen como Workdir /app
```dockerfile
FROM node:22

WORKDIR /app 

RUN apt-get update \
 && apt-get install -y --no-install-recommends git \
 && rm -rf /var/lib/apt/lists/*

RUN git clone https://github.com/ruanyf/webpack-demos.git

# wordir ahora hace de cd, nos movemos a la carpeta webpack-demos
WORKDIR /app/webpack-demos 
RUN npm install

WORKDIR /app/webpack-demos/demo06
RUN npm install && npm run build

#creamos esta carpeta que es la que usamos en volumes del contenedor
RUN mkdir -p /build/demo

#esto no es necesario realmente, porque se pone en el compose
EXPOSE 5173

#importante, aqui faltaba añadir el index.html para que se viera el contenido
CMD ["sh", "-c", "npm run build && cp -f bundle.js /build/demo/bundle.js && cp -f index.html /build/demo/index.html"]
```

>[!warning] OJO! si no pide nada, por defecto es dev no build, esto hace que se quede encendido el contenedor para poder desarrollar, con el build no se podria modificar nada de dentro.

```yaml
#1 - Createme en vite un proyecto vue

#2 - Cread un docker compose para este proyecto (sin build)

#3 - Instalad scss en el proyecto (ir al contenedor anterior y ejecutar npm install -D sass)

#4 - Cread un contendor de un proyecto de github (con build)

#5 - Crea un contenedor con apache que mire los proyectos anteriores (sus builds)

  

name: "examen-prueba"
services:
 vite:
  build:
   context: .
   dockerfile: ./vite/dockerfile
  tty: true
  stdin_open: true
  working_dir: /app
  volumes:
    - ./vite:/app
  ports:
    - "8082:5173"
      
 demo6:
  build:
   context: .
   dockerfile: ./demo/dockerfile
  tty: true
  stdin_open: true
  working_dir: /app/webpack-demos/demo06
  volumes:
    - ./demo:/build/demo
  ports:
    - "8083:5173"

 apache:
  build:
   context: .
   dockerfile: ./apache/dockerfile
  tty: true
  stdin_open: true
  volumes:
   - ./vite/dist:/var/www/vite
   - ./demo:/var/www/demo
  ports:
    - "8084:80"
    - "8085:81"
```
Vite:
```dockerfile
FROM node:24

RUN apt update

COPY ./vite/package.json .

RUN npm install -D sass 

CMD ["sh", "-c", "npm run build && npm run dev"]
```
Apache:
```dockerfile
FROM httpd:2.4

RUN mkdir -p /var/www/vite /var/www/demo

COPY apache/httpd.conf /usr/local/apache2/conf/httpd.conf
COPY apache/httpd-vhosts.conf /usr/local/apache2/conf/extra/httpd-vhosts.conf
```
http-vhosts.conf
```conf
<VirtualHost *:81>

    ServerName localhost
    DocumentRoot "/var/www/vite"  

    <Directory "/var/www/vite">
        Options -Indexes +FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>
    
    ErrorLog "logs/vite-error.log"
    CustomLog "logs/vite-access.log" combined

</VirtualHost>

<VirtualHost *:80>

    ServerName localhost
    DocumentRoot "/var/www/demo"

    <Directory "/var/www/demo">
        Options -Indexes +FollowSymLinks
        AllowOverride None
        Require all granted

    </Directory>

    ErrorLog "logs/demo-error.log"
    CustomLog "logs/demo-access.log" combined

</VirtualHost>
```

Demo06
```dockerfile
FROM node:22

WORKDIR /app

RUN apt-get update \
 && apt-get install -y --no-install-recommends git \
 && rm -rf /var/lib/apt/lists/*

RUN git clone https://github.com/ruanyf/webpack-demos.git  

WORKDIR /app/webpack-demos

RUN npm install 

WORKDIR /app/webpack-demos/demo06

RUN npm install && npm run build

RUN mkdir -p /build/demo

CMD ["sh", "-c", "npm run build && cp -f bundle.js /build/demo/bundle.js && cp -f index.html /build/demo/index.html"]
```