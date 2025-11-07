# A) Con `docker run` (estilo profesor)

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
docker cp llista-camisetes/. web:/usr/local/apache2/htdocs
docker cp images-camisetes/. web:/usr/local/apache2/htdocs/images
# (opcional)
docker exec -it web bash
```

> `DocumentRoot (httpd)`: `/usr/local/apache2/htdocs`  
> `docker cp` deja tu web **dentro del contenedor** (sin volúmenes).

---

### 2.2 PHP con Apache (`php:apache`)

**Opción A (montar carpeta del host, ejemplo Windows del profe):**
```powershell
docker run -d -p 80:80 --name php ^
  -v "C:\Users\Alexandra\...\PHP":/var/www/html ^
  php:7.2-apache
```

**Opción B (copiar el código al contenedor):**
```bash
docker run -d --name php -p 8000:80 php:apache
docker exec -it php bash -lc "echo '<?php phpinfo(); ?>' > /var/www/html/info.php"

git clone -b json https://github.com/antonimoragues/llibres llibres-json
docker cp llibres-json/.    php:/var/www/html
docker cp imatges-llibres/. php:/var/www/html/images
```

> `DocumentRoot (php:apache)`: `/var/www/html` (**distinto** de `httpd`).

---

### 2.3 MariaDB + importar datos
```bash
docker run -d --name db -e MARIADB_ROOT_PASSWORD=secret mariadb
docker cp books.sql db:/tmp
docker exec -it db bash
mariadb -uroot -psecret -e "CREATE DATABASE IF NOT EXISTS llibres CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
mariadb -uroot -psecret llibres < /tmp/books.sql
mariadb -uroot -psecret -e "USE llibres; SHOW TABLES;"
```

- `-e MARIADB_ROOT_PASSWORD=secret`: inicializa **password de root**.
- Comprobación rápida: `SHOW TABLES;`.

---

### 2.4 Extensiones PHP (MySQL)
```bash
docker exec -it php bash -lc "docker-php-ext-install mysqli pdo pdo_mysql && apache2ctl -k graceful"
```

- `docker-php-ext-install`: instalador de extensiones en imágenes **oficiales** PHP.
- `apache2ctl -k graceful`: recarga sin cortar conexiones.

---

### 2.5 Redes Docker (DNS interno por nombre)
```bash
docker network create llibres
docker run -d --name db --network llibres -e MARIADB_ROOT_PASSWORD=secret mariadb
docker run -d --network llibres -p 8000:80 --name admin -e PMA_HOST=db phpmyadmin
docker run -d --network llibres --name web -p 8001:80 php:apache
```

- En la **misma red**, `db` se resuelve por **nombre**.
- `PMA_HOST=db` le dice a phpMyAdmin dónde está la base de datos.

---

### 2.6 Volumen para persistencia de MariaDB
```bash
docker volume create db-llibres
docker run -d --name db --network llibres \
  -e MARIADB_ROOT_PASSWORD=secret \
  -v db-llibres:/var/lib/mysql \
  mariadb
```

> Si borras el contenedor, los **datos** quedan en el volumen `db-llibres`.

---

### 2.7 VirtualHost y SSL **dentro** del contenedor (laboratorio)

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

### 2.8 Limpieza
```bash
docker ps -a
docker rm -f web php db admin apache || true
```

---

# B) Con `docker compose` (mismo flujo empaquetado)

**Mismas ideas** pero declaradas en un archivo.  
Comandos clave: `docker compose up -d`, `ps`, `logs`, `exec`, `cp`, `down`.

## 1) Crear `docker-compose.yml`
```bash
cat > docker-compose.yml <<'YML'
services:
  db:
    image: mariadb
    container_name: db
    environment:
      - MARIADB_ROOT_PASSWORD=secret
    volumes:
      - db-llibres:/var/lib/mysql
    networks:
      - llibres

  admin:
    image: phpmyadmin
    container_name: admin
    environment:
      - PMA_HOST=db
    ports:
      - "8000:80"
    depends_on:
      - db
    networks:
      - llibres

  web:
    image: php:apache
    container_name: web
    ports:
      - "8001:80"
    networks:
      - llibres

  # Opcional: httpd para sitio estático en 80
  httpd:
    image: httpd
    container_name: web-static
    ports:
      - "80:80"
    networks:
      - llibres

volumes:
  db-llibres:

networks:
  llibres:
    driver: bridge
YML
```

## 2) Levantar, ver y parar
```bash
docker compose up -d
docker compose ps
docker compose logs -f web
docker compose down        # para (sin borrar volúmenes)
docker compose down -v     # para y borra volúmenes
```

## 3) Copias y comandos (igual que con `run`)

**Copiar web estática a `httpd`:**
```bash
docker cp llista-camisetes/.   web-static:/usr/local/apache2/htdocs
docker cp images-camisetes/.   web-static:/usr/local/apache2/htdocs/images
```

**Copiar app PHP:**
```bash
docker cp llibres-json/.       web:/var/www/html
docker cp imatges-llibres/.    web:/var/www/html/images
```

**Importar SQL:**
```bash
docker cp books.sql            db:/tmp
docker compose exec db bash -lc \
  "mariadb -uroot -psecret -e 'CREATE DATABASE IF NOT EXISTS llibres CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;' && \
   mariadb -uroot -psecret llibres < /tmp/books.sql && \
   mariadb -uroot -psecret -e 'USE llibres; SHOW TABLES;'"
```

**Instalar extensiones PHP:**
```bash
docker compose exec web bash -lc "docker-php-ext-install mysqli pdo pdo_mysql && apache2ctl -k graceful"
```

> Para VirtualHosts/SSL usa los **mismos comandos** que en la sección A, pero con  
> `docker compose exec web ...` en lugar de `docker exec apache ...`.

---

## Chuleta rápida — DocumentRoot por imagen

| Imagen     | DocumentRoot                     |
|-----------:|----------------------------------|
| `httpd`    | `/usr/local/apache2/htdocs`      |
| `php:apache` | `/var/www/html`                |