---
title: Stack "llibres" con Docker Compose — Guía paso a paso (Obsidian)
tags: [docker, docker-compose, httpd, php, mariadb, phpmyadmin, scp]
---

# Docker Compose Ejemplo
## Comandos básicos
```bash
docker compose up -d
docker compose ps
docker compose logs -f web
docker compose down        # para (sin borrar volúmenes)
docker compose down -v     # para y borra volúmenes
```


> [!summary] Objetivo
> Preparar un entorno reproducible con **httpd** (estático), **php:apache**, **MariaDB** con import inicial y **phpMyAdmin**, usando **variables** en un único archivo y **Dockerfiles** por servicio.


# Estructura de proyecto `~/docker-web`

> [!summary] Visión general
> Árbol de directorios listo para copiar/pegar en Obsidian con breve descripción de cada carpeta/archivo clave.

```text
~/compose-llibres
├── apache/
│   ├── sites-available/
│   └── sites-enabled/
│       └── www.llibres.com.conf
├── certs/
│   ├── certs/
│   │   ├── www.llibres.com.crt
│   │   ├── www.camisetes.com.crt
│   │   └── www.db.com.crt
│   └── private/
│       ├── www.llibres.com.key
│       ├── www.camisetes.com.key
│       └── www.db.com.key
├── llista-camisetes/
│   └── index.html
├── llibres-db/
│   └── index.php
├── phpmyadmin/
│   └── (opcional)
├── passwords/
│   ├── mysql.env
│   ├── pma.env
│   └── web.env
├── web/
│   └── Dockerfile
└── docker-compose.yml
```

---

## 0) Preparación (si falta `git`)
```bash
#comprobamos: 
git --version 

# Si no sale nada, no esta instalado y debemos hacer:
sudo apt update
sudo apt install -y git
git config --global user.name "Alexandra1055"
git config --global user.email "agonzalez1055@alumnes.politecnicllevant.cat"
```

---

## 1) Estructura de carpetas + clones

```bash
# Carpeta del proyecto
mkdir -p ~/compose-llibres/{web,php,mysql-php,db/init,passwords}
mkdir -p ~/home/examen/{asix,daw,gateway}

cd ~/compose-llibres

# 2.1 Sitio estático (httpd)
cd web
git clone https://github.com/antonimoragues/botiga-examen.git
cd ..

# 2.2 PHP básico (php:apache) con llibres-json
cd php
git clone -b json https://github.com/antonimoragues/llibres llibres-json
cd ..

# 2.3/2.4 PHP con MySQLi/PDO + DB (código y SQL)
cd mysql-php
git clone -b database https://github.com/antonimoragues/llibres llibres-db
cd ..

# Copia el SQL de import a la carpeta de init de MariaDB
cp ./mysql-php/llibres-db/books.sql ./db/init/books.sql

# Para meter los dockerfile
mkdir -p ~/docker-web/web
```

> [!tip] Coloca imágenes **antes** de construir
> - Estático: `~/compose-llibres/web/llista-camisetes/images/`  
> - PHP json: `~/compose-llibres/php/llibres-json/images/`  
> - PHP + DB: `~/compose-llibres/mysql-php/llibres-db/images/`

---

## 2) Archivo de variables `password`
```bash
cat > ~/botiga-examen/passwords/password <<'EOF'
# --- MariaDB ---
MARIADB_ROOT_PASSWORD=secret
MARIADB_DATABASE=llibres
# Si solo usarás root, no declares MARIADB_USER/PASSWORD extra
# !!!--- MySQL ---
MYSQL_ROOT_PASSWORD=secret
MYSQL_DATABASE=llibres
# Si solo usarás root, no declares MARIADB_USER/PASSWORD extra
# (si necesitas usuario de app)
# MYSQL_USER=appuser

# --- App PHP que se conecta a MariaDB ---
DB_HOST=db
DB_NAME=llibres
DB_USER=root
DB_PASS=secret

# --- phpMyAdmin ---
PMA_HOST=db
PMA_USER=root
PMA_PASSWORD=secret
EOF

# Opcional: restringe permisos
chmod 600 ~/compose-llibres/passwords/password
```
>[!warning] OJO! si es mariadb o mysql

> [!note]
> `env_file` carga estas variables **solo** en los servicios que lo declaran; las no usadas **se ignoran**.

---

## 3) Dockerfiles

Creamos el archivo y su contenido: 
```bash
cat > ~/compose-llibres/web/Dockerfile <<'DOCKERFILE'
 # aqui meto lo de abajo: 
 DOCKERFILE
 
 # para cada dockerfile su carpeta: web, php y mysql-php
```


### `web/Dockerfile` (httpd + estático)
```dockerfile
# ~/compose-llibres/web/Dockerfile
FROM httpd:2.4
# Mantiene DocumentRoot de httpd: /usr/local/apache2/htdocs
COPY llista-camisetes/ /usr/local/apache2/htdocs/
```

### `php/Dockerfile` (php:apache + phpinfo + llibres-json)
```dockerfile
cat > ~/home/examen/asix/src/Dockerfile <<'DOCKERFILE'
	FROM php:apache
	RUN bash -lc "echo '<?php phpinfo(); ?>' > /var/www/html/info.php"
 DOCKERFILE
# ~/compose-llibres/php/Dockerfile
FROM php:apache
# Mantiene DocumentRoot de php:apache: /var/www/html
RUN bash -lc "echo '<?php phpinfo(); ?>' > /var/www/html/info.php"
COPY llibres-json/ /var/www/html/
```

### `mysql-php/Dockerfile` (php:apache + extensiones MySQL + app)
```dockerfile
# ~/compose-llibres/mysql-php/Dockerfile
FROM php:apache
RUN docker-php-ext-install mysqli pdo pdo_mysql
# Copia la app que usa DB
COPY llibres-db/ /var/www/html/
# La app deberá leer getenv('DB_HOST'), getenv('DB_NAME'), etc.
```

---

## 4) `docker-compose.yml` (con `env_file: passwords/password`)

Creamos el archivo y su contenido: 
```bash
cat > ~/compose-llibres/docker-compose.yml <<'YML'
 #aqui meto lo de abajo: version....
 YML
```

```yaml
# ~/compose-llibres/docker-compose.yml
version: "3.8"

name: llibres-stack

services:
  # 2.1: Sitio estático con httpd
  web:
    build:
      context: ./web
      dockerfile: Dockerfile
    container_name: web
    ports:
      - "80:80"
    networks:
      - llibres

  # 2.2: PHP básico (phpinfo + llibres-json)
  php:
    build:
      context: ./php
      dockerfile: Dockerfile
    container_name: php
    ports:
      - "8000:80"
    networks:
      - llibres

  # 2.3 y 2.6: MariaDB con volumen persistente e import inicial
  db:
    image: mariadb:latest
    container_name: db
    env_file:
      - ./passwords/password
    volumes:
      - db-llibres:/var/lib/mysql
      - ./db/init/books.sql:/docker-entrypoint-initdb.d/books.sql:ro
    networks:
      - llibres
# 2.3 y 2.6: MySQL
  db:
    image: mysql:8
    container_name: db
    env_file:
      - ./passwords/password
    volumes:
      - db-llibres:/var/lib/mysql
      - ./db/init/books.sql:/docker-entrypoint-initdb.d/books.sql:ro
    networks:
      - llibres

  # 2.4: PHP con extensiones MySQL y app que conecta a db
  mysql-php:
    build:
      context: ./mysql-php
      dockerfile: Dockerfile
    container_name: mysql-php
    env_file:
      - ./passwords/password
    depends_on:
      - db
    ports:
      - "8001:80"
    networks:
      - llibres

  # Utilidad: phpMyAdmin para revisar tablas
  phpmyadmin:
    image: phpmyadmin
    container_name: phpmyadmin
    env_file:
      - ./passwords/password
    depends_on:
      - db
    ports:
      - "8083:80"
    networks:
      - llibres

networks:
  llibres:
    name: llibres

volumes:
  db-llibres:
    name: db-llibres
```

>[!warning] OJO! si es mariadb o mysql

---

## 5) Levantar y comprobar

```bash
cd ~/compose-llibres
docker compose up -d --build
docker compose ps
```

- **Estático:** `http://IP_PUBLICA/`  
- **PHP básico:** `http://IP_PUBLICA:8000/info.php` ( + páginas de `llibres-json`)  
- **App PHP + DB:** `http://IP_PUBLICA:8001/`  
- **phpMyAdmin:** `http://IP_PUBLICA:8083/` (login `root` / `secret`, host `db`)

**Reimportar SQL** (si cambias `books.sql`):
```bash
docker compose down
docker volume rm db-llibres
docker compose up -d --build
```

---

## 6) Pasar imágenes por **SCP** (opcional) y **reconstruir**

**Desde Windows (ajusta clave e IP):**
```powershell
# Estático (httpd)
scp -i "agonzalez" -r "/C:\Users\Alexandra\Documents\DAW25-26\DesplegamentWeb\SSH/imatges.zip" ` root@ifc.politecnicllevant.cat:~/var/www/html/images/

# PHP json
scp -i "prueba.pem" -r "/d/Documentos/DAW25-26/Desplegament/imatges/." `
  admin@IP_PUBLICA:~/compose-llibres/php/llibres-json/images/

# PHP + DB
scp -i "prueba.pem" -r "/d/Documentos/DAW25-26/Desplegament/imatges/." `
  admin@IP_PUBLICA:~/compose-llibres/mysql-php/llibres-db/images/
```

**Reconstruir servicios que empaquetan contenido:**
```bash
cd ~/compose-llibres
docker compose build web php mysql-php
docker compose up -d
```

---

##### db.com.conf

```bash
<VirtualHost *:80>  
    ServerName db.com  
    ServerAlias www.db.com  
    Redirect permanent / https://db.com/  
  
    ProxyPreserveHost On  
    ProxyPass / http://phpmyadmin:80/  
    ProxyPassReverse / http://phpmyadmin:80/  
  
    ErrorLog ${APACHE_LOG_DIR}/error_ssl.log  
    CustomLog ${APACHE_LOG_DIR}/access_ssl.log combined  
</VirtualHost>  
  
<VirtualHost *:443>  
    ServerName www.db.com  
    DocumentRoot /var/www/db.com  
  
    SSLEngine on  
    SSLCertificateFile /etc/ssl/db/db.crt  
    SSLCertificateKeyFile /etc/ssl/db/db.key  
  
    ProxyPreserveHost On  
    ProxyPass / http://asix-img:80/  
    ProxyPassReverse / http://asix-img:80/  
  
    ErrorLog ${APACHE_LOG_DIR}/error_ssl.log  
    CustomLog ${APACHE_LOG_DIR}/access_ssl.log combined  
  
</VirtualHost>
```

```bash
cat << 'APACHE' | tee /etc/apache2/sites-available/www.camisetes_ssl.com.conf > /dev/null  
<VirtualHost *:443>  
ServerName www.asix.net  
ServerAlias asix.net  
DocumentRoot /var/www/asix.net  
  
SSLEngine On  
SSLCertificateFile /etc/ssl/certs/asix.key  
SSLCertificateKeyFile /etc/ssl/certs/asix.crt  
  
<Directory /var/www/asix.net>  
AllowOverride All  
Require all granted  
DirectoryIndex llista.php  
Options +Indexes  
</Directory>  
  
ErrorLog ${APACHE_LOG_DIR}/camisetes_error_ssl.log  
CustomLog ${APACHE_LOG_DIR}/camisetes_access_ssl.log combined  
  
DirectoryIndex llista.php index.php  
</VirtualHost>  
APACHE
```

## Consejos

- No subas `passwords/password` a repositorios (**añádelo** a `.gitignore`).
- Si tu `list.php` **no** lee variables de entorno, fija el host de conexión a `'db'`.  
  **Mejor:** usa `getenv('DB_HOST')`, `getenv('DB_NAME')`, etc. (vienen de `env_file`).

> [!check] Chuleta rápida de DocumentRoot
> - `httpd` → `/usr/local/apache2/htdocs`  
> - `php:apache` → `/var/www/html`

scp -i "agonzalez" -P 20722 "/C:\Users\Alexandra\Documents\DAW25-26\DesplegamentWeb\SSH\imatges.zip" root@ifc.politecnicllevant.cat:/var/www/html/images/