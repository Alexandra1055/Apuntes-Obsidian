# A) Con `docker run`

## 1) Comandos base y significado

**Lanzar contenedor (descarga la imagen si no existe)**  
```bash
docker run -d -p <host:cont> --name <nombre> <imagen>
```

- `-d`: ejecuta en **segundo plano** (*detached*).
- `-p 80:80`: **mapea puertos** → puerto `80` del host a `80` del contenedor.
- `--name web`: asigna un **nombre** (más cómodo que el ID).
- `<imagen>`: nombre de imagen (`httpd`, `php:apache`, `mariadb`, …).

> [!warning] Ojo con el **orden**
> Los *flags* (`-d -p --name ...`) van **antes** de la imagen.  
> **Ejemplo correcto:**  
> ```bash
> docker run -d --name apacheAle -p 8001:80 httpd:2.4
> ```

### Inspección y control
```bash
docker ps              # contenedores en ejecución
docker ps -a           # todos (ejecución + detenidos)
docker ps -q           # solo IDs de los que están en ejecución
docker ps -a -q        # solo IDs de todos
docker logs <nombre>   # ver logs del contenedor
docker stop <nombre>   # parar
docker rm <nombre>     # eliminar (si está parado)
docker rm -f <nom>     # forzar eliminación aunque esté arrancado
docker rm -f $(docker ps -a -q)  # limpiar todos (con cuidado)
docker start <nombre>  #para iniciar los contendores parados
docker rmi <nombre_imagen>       #para eliminar la imagen
```

### Ejecutar comandos dentro del contenedor
```bash
docker exec -it <nombre> bash                # sesión interactiva
docker exec -it <nombre> ls /ruta            # listar
docker exec -it <nombre> bash -lc "comando"  # login shell + comando largo
```

- `-i/-t`: **interactivo** + pseudo-**TTY** (útil para `bash`, `nano`, etc.).
- `bash -lc "..."`: abre shell, **carga entorno** y ejecuta una cadena.

### Copiar ficheros
```bash
docker cp <origen_host> <contenedor>:/ruta_destino
docker cp <contenedor>:/ruta_origen <destino_host>
```

### Variables de entorno, volúmenes y red (estilo profe)
```bash
docker run -d --name db -e MARIADB_ROOT_PASSWORD=secret mariadb    # -e: variable entorno
docker volume create db-llibres                                    # volumen nombrado
docker run -d --name db -v db-llibres:/var/lib/mysql mariadb       # -v: monta volumen
docker network create llibres                                      # red bridge
docker run -d --network llibres --name web -p 8001:80 php:apache   # unir a red
```

---

## 2) Casos del ejercicio (paso a paso)

### 2.1 Sitio estático con Apache `httpd`
```bash
docker run -d -p 80:80 --name web httpd
docker ps
docker exec -it web ls /usr/local/apache2/htdocs
```

 >[!warning] Si el git no esta instalado:
```bash
sudo apt install git -y
git config --global user.name "Alexandra1055"
git config --global user.email "agonzalez1055@alumnes.politecnicllevant.cat"

```

```bash
git clone https://github.com/antonimoragues/llista-camisetes.git
docker cp llista-camisetes/. web:/usr/local/apache2/htdocs
# (comprobamos)
http://ip-publica
```

Ahora descargaríamos las imágenes (si nos lo pide) y las pasamos a la maquina con:
```bash
scp -i "prueba.pem" -r "/d/Documentos/DAW25-26/Desplegament/llista-camisetes/images/." admin@44.200.227.108:~/llista-camisetes/images/
#admin@ip-publica de la maquina
#-r "ruta completa con /." recursivamente que copie lo que hay dentro
docker cp images/. web:/usr/local/apache2/htdocs/images
#Comprobamos -> (actualizamos el navegador y saldran las imagenes)
```

> `DocumentRoot (httpd)`: `/usr/local/apache2/htdocs`  
> `docker cp` deja tu web **dentro del contenedor** (sin volúmenes).

---

### 2.2 PHP con Apache (`php:apache`)

#### Copiar el código al contenedor:
```bash
docker run -d --name php -p 8000:80 php:apache
docker run -d --name asix-img -p 81:80 php:apache
docker exec -it php bash -lc "echo '<?php phpinfo(); ?>' > /var/www/html/info.php"
docker exec -it asix-img bash -lc "echo '<?php phpinfo(); ?>' > /var/www/html/info.php"
#comprobamos: http://ip-publica:8000/info.php

git clone -b json https://github.com/antonimoragues/daw.git daw
docker cp llibres-json/.    php:/var/www/html
```

Como antes, descargamos y pegamos las imágenes:
```bash
scp -i "prueba.pem" -r "/d/Documentos/DAW25-26/Desplegament/imatges/." admin@44.200.227.108:~/llibres-json/images/

docker cp images/. php:/var/www/html/images
#comprobamos: http://ip-publica:8000/list.php
```

> `DocumentRoot (php:apache)`: `/var/www/html` (**distinto** de `httpd`).

---

### 2.3 MariaDB + importar datos
```bash
docker run -d --name db -e MARIADB_ROOT_PASSWORD=secret mariadb

git clone -b database https://github.com/antonimoragues/llibres llibres-db


docker cp books.sql db:/tmp
docker exec -it db bash
mariadb -uroot -psecret -e "CREATE DATABASE IF NOT EXISTS llibres CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
mariadb -uroot -psecret llibres < /tmp/books.sql
mariadb -uroot -psecret -e "USE llibres; SHOW TABLES;"
```

- `-e MARIADB_ROOT_PASSWORD=secret`: inicializa **password de root**.
- Comprobación rápida: `SHOW TABLES;`.

>[!warning] OJO! si es mariadb o mysql
### 2.3.1 MYSQL
Si en lugar de mariaDB es MySQL
```bash

# (ahora) MySQL:
docker run -d --name db -e MYSQL_ROOT_PASSWORD=secret mysql:8

# (antes) MariaDB con volumen:
# docker run -d --name db -e MARIADB_ROOT_PASSWORD=secret mariadb

# Descargar los SQL (igual que antes)
git clone -b database https://github.com/antonimoragues/llibres llibres-db

# Copiar el SQL dentro del contenedor
docker cp llibres-db/books.sql db:/tmp

# Entrar al contenedor y cargar la BD
docker exec -it db bash

# Crear la base de datos con UTF-8 moderno
mysql -uroot -psecret -e "CREATE DATABASE IF NOT EXISTS llibres CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"

# Importar datos
mysql -uroot -psecret llibres < /tmp/books.sql

# Comprobación
mysql -uroot -psecret -e "USE llibres; SHOW TABLES;"

# salir si quieres
exit

# Red mas abajo (quedaria igual el comando):
docker network connect llibres db

```

---

### 2.4 Extensiones PHP (MySQL)
```bash
docker run -d --name mysql-php -p 8001:80 php:apache
docker exec -it mysql-php bash -lc "docker-php-ext-install mysqli pdo pdo_mysql && apache2ctl -k graceful"
```
Editamos el Host de la base de datos con el nombre del contenedor donde tenemos la base de datos:
```bash
#instalamos nano
sudo apt update
sudo apt install -y nano

#editamos el archivo
sudo nano list.php #dbHost = getenv('DB_HOST') ?: 'db';
#comprobamos (cat list.php)

docker cp llibres-db/. mysql-php:/var/www/html
```

- `docker-php-ext-install`: instalador de extensiones en imágenes **oficiales** PHP.
- `apache2ctl -k graceful`: recarga sin cortar conexiones.

---

### 2.5 Redes Docker (DNS interno por nombre)

```bash
docker network create llibres
docker network connect llibres db
docker network connect llibres mysql-php

docker network remove llibres #para eliminar la network
```

- En la **misma red**, `db` se resuelve por **nombre**.
- `PMA_HOST=db` le dice a phpMyAdmin dónde está la base de datos.
-
>Comprobaciones rápidas de network:
```bash
# ¿Existe la red?
docker network ls | grep llibres

# ¿Está 'db' en esa red?
docker inspect -f '{{json .NetworkSettings.Networks}}' db | jq

# (o sin jq)
docker inspect -f '{{range $k,$v := .NetworkSettings.Networks}}{{$k}} {{end}}' db
```

---
### 2.6 PHPMyAdmin
```bash
docker run -d --name phpmyadmin \
  --network llibres \
  -p 8083:80 \
  -e PMA_HOST=db \
  -e PMA_USER=root \
  -e PMA_PASSWORD=secret \
  phpmyadmin/phpmyadmin
  
  #opcional: se podria añadir para que apunte al puerto mysql
  -e PMA_PORT=3306 \
  
  #comprobamos: http://IP-PUBLICA:8083
```

---
### 2.7 Volumen para persistencia de MariaDB
```bash
docker volume create db-llibres
docker run -d --name db --network llibres \
  -e MARIADB_ROOT_PASSWORD=secret \
  -v db-llibres:/var/lib/mysql \
  mariadb
```

> Si borras el contenedor, los **datos** quedan en el volumen `db-llibres`.

---

### 2.8 VirtualHost y SSL **dentro** del contenedor (laboratorio)

**Crear/editar VirtualHost:**
```bash
docker exec -it apache bash
cd /etc/apache2/sites-available
cp 000-default.conf camisetes.com.conf
exit
docker exec -it apache grep -v "#" /etc/apache2/sites-available/camisetes.com.conf > camisetes.com.conf
# (EDITA LOCAL) y vuelve a meterlo:
docker cp camisetes.com.conf apache:/etc/apache2/sites-available/
docker exec -it apache bash -lc "a2ensite camisetes.com.conf && apachectl -k restart"
```

**Habilitar SSL + certificados self-signed (pruebas):**
```bash
docker exec -it apache bash -lc "a2enmod ssl || true"
docker exec -it apache bash
cd /tmp
openssl genrsa -des3 -out server.key 2048
openssl rsa -in server.key -out server.key.insecure
mv server.key server.key.secure && mv server.key.insecure server.key
openssl req -new -key server.key -out server.csr        # Common Name: www.camisetes.com
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
mkdir -p /etc/apache2/certs
cp /tmp/server.crt /etc/apache2/certs/
cp /tmp/server.key /etc/apache2/certs/
exit
docker exec -it apache grep -v "#" /etc/apache2/sites-enabled/default-ssl.conf > default-ssl.conf
# (EDITA LOCAL: rutas certs) y vuelve a copiarlo
docker cp default-ssl.conf apache:/etc/apache2/sites-enabled/
docker exec -it apache bash -lc "apachectl -k restart"
```

- `a2ensite` / `a2enmod`: habilitan **sitios/módulos** (Debian/Ubuntu).
- `openssl`: genera clave, CSR y **certificado autofirmado** (solo laboratorio).

---

### 2.9 Limpieza
```bash
docker ps -a
docker rm -f web php db admin apache || true
```


---

## Chuleta rápida — DocumentRoot por imagen
Para saber cual es el directorio que usa la imagen:
```bash
docker exec -it <nombre_contenedor> pwd
```

|       Imagen | DocumentRoot                |
| -----------: | --------------------------- |
|      `httpd` | `/usr/local/apache2/htdocs` |
| `php:apache` | `/var/www/html`             |
PROXMOX
**FALTA REDIRECT y ALIAS

ssh -i agonzalez root@10.100.78.207
//entramos en el insti

ssh -i agonzalez -p 20722 [root@ifc.politecnicllevant.cat](mailto:root@ifc.politecnicllevant.cat)
//para entrar a proxmos desde cada

IMPORTANTE:
Si falla al intentar entrar, usar este comando
ssh-keygen -R 10.100.78.207

Ahora puedo entrar desde:
ssh proxmox-cable