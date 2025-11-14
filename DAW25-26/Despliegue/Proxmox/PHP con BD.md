### 1) Instalar MariaDB y extensión PHP–MySQL
```bash
apt install -y mariadb-server php-mysql

systemctl enable --now mariadb

#comprobamos version de mysql
mysql -e "SELECT VERSION();" 
```

### 2) Crear BD y usuario (DB: `llibres`, user: `root`, pass: `secret`)

```bash
mysql -e "CREATE DATABASE llibres DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
mysql -e "CREATE USER 'root'@'localhost' IDENTIFIED BY 'secret';"
mysql -e "GRANT ALL PRIVILEGES ON llibres.* TO 'root'@'localhost'; FLUSH PRIVILEGES;"
```

### 3) Desplegar la rama database del repo

```bash
mkdir -p /var/www/html/llibres-db

git clone -b database --single-branch https://github.com/antonimoragues/llibres.git /var/www/html/llibres-db

#Importar el esquema (archivo `books.sql` del repo)
mysql -uroot -p'secret' llibres < /var/www/html/llibres-db/books.sql

```

>[!note] Debemos cambiar lo que hay dentro del book.php para poder accedes:
```php
$dbHost = getenv('DB_HOST') ?: '127.0.0.1';
$dbPort = getenv('DB_PORT') ?: '3306';
$dbName = getenv('DB_DATABASE') ?: 'llibres';
$dbUser = getenv('DB_USERNAME') ?: 'root';
$dbPass = getenv('DB_PASSWORD') ?: 'secret';
```

>[!warning] Es importante resetear el apache para que se vean los cambios hechos:
>	systemctl restart apache2

### 4) Cambiamos los permisos
```bash
chown -R www-data:www-data /var/www/html/llibres-db 
find /var/www/html/llibres-db -type d -exec chmod 755 {} \; 
find /var/www/html/llibres-db -type f -exec chmod 644 {} \;
```
### 5) Pasar las imágenes de json a db

```bash
# nos movemos a json images
cp -r . /var/www/html/llibres-db/images/  

# Nos movemos a db y comprobamos 
cd /var/www/html/llibres-db/images/
ls
```

### 6) Priorizar list.php y index.php

Abre en el navegador:

```bash
cd /etc/apache2/mods-enabled
nano dif.conf  #dif.conf a nivel global 
cd /etc/apache2/sites-available
nano 000-default.conf #alternativa no global
#añadimos: **DirectoryIndex list.php index.php**
```
### 7) Desactivar listado de directorios (quitar `Indexes`)

Edita el bloque `<Directory /var/www/>` en `/etc/apache2/apache2.conf`:

```bash
nano /etc/apache2/apache2.conf
```

Asegúrate de que quede **sin** `Indexes`, por ejemplo:

```apache
<Directory /var/www/>
        Options FollowSymLinks
        AllowOverride None
        Require all granted
</Directory>
#**Options Indexes FollowSymLinks //borramos Indexes**
```

>[!warning] Importante resetear el apache:
>	systemctl restart apache2