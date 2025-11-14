### 1) Preparar los DocumentRoot

```bash
# Comprueba lo que tienes
ls -l /var/www/html

# Copia tus webs a /var/www/ manteniendo permisos y fechas
cp -a /var/www/html/llista-camisetes /var/www/
cp -a /var/www/html/llibres-db       /var/www/

```

# 2) Crear vhosts HTTP (puerto 80)

Crearemos 3 sitios:

- `www.camisetes.com` → `/var/www/llista-camisetes`
    
- `www.llibres.com` → `/var/www/llibres-db`
    
- `www.db.com` → `/usr/share/phpmyadmin` (si existe)
    

> Usa **heredocs** con `tee` para no abrir un editor. Corrijo la sintaxis y variables de log.

```bash
tee /etc/apache2/sites-available/www.camisetes.com.conf > /dev/null <<'APACHE'
<VirtualHost *:80>
    ServerName www.camisetes.com
    ServerAlias camisetes.com
    DocumentRoot /var/www/llista-camisetes

    DirectoryIndex index.html index.php
    ErrorLog  ${APACHE_LOG_DIR}/camisetes_error.log
    CustomLog ${APACHE_LOG_DIR}/camisetes_access.log combined

    <Directory /var/www/llista-camisetes>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
APACHE
# cd /etc/apache2//sites-available/ asi podemos hacer ls y ver que se crean
tee /etc/apache2/sites-available/www.llibres.com.conf > /dev/null <<'APACHE'
<VirtualHost *:80>
    ServerName www.llibres.com
    ServerAlias llibres.com
    DocumentRoot /var/www/llibres-db

    # Prioriza list.php como página por defecto
    DirectoryIndex list.php index.php index.html
    ErrorLog  ${APACHE_LOG_DIR}/llibres_error.log
    CustomLog ${APACHE_LOG_DIR}/llibres_access.log combined

    <Directory /var/www/llibres-db>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
APACHE
if [ -d /usr/share/phpmyadmin ]; then
tee /etc/apache2/sites-available/www.db.com.conf > /dev/null <<'APACHE'
<VirtualHost *:80>
    ServerName www.db.com
    ServerAlias db.com
    DocumentRoot /usr/share/phpmyadmin

    DirectoryIndex index.php
    ErrorLog  ${APACHE_LOG_DIR}/db_error.log
    CustomLog ${APACHE_LOG_DIR}/db_access.log combined

    <Directory /usr/share/phpmyadmin>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
APACHE
fi
```
Habilita los sitios y (opcional) deshabilita el `000-default`:
```bash
a2ensite www.camisetes.com.conf
a2ensite www.llibres.com.conf
[ -f /etc/apache2/sites-available/www.db.com.conf ] && a2ensite www.db.com.conf

# Opcional: desactivar sitio por defecto para evitar solapes
a2dissite 000-default.conf || true

apachectl configtest
systemctl reload apache2

```

### 3) HTTPS con certificados autosignados (uno por dominio)

Activa SSL:

```bash
a2enmod ssl
```

Genera claves y certificados **válidos 1 año**. Guarda la **key** en `/etc/ssl/private` (más restrictivo) y el **crt** en `/etc/ssl/certs`.

```bash
cd /etc/ssl/private
# Repite para cada dominio
for DOMAIN in www.camisetes.com www.llibres.com www.db.com; do
  # clave privada
  openssl genrsa -out /etc/ssl/private/${DOMAIN}.key 2048
  chmod 600 /etc/ssl/private/${DOMAIN}.key

  # CSR + CRT autosignado (sin CSR intermedio, con -subj)
  openssl req -x509 -new -nodes -sha256 -days 365 \
    -key /etc/ssl/private/${DOMAIN}.key \
    -out /etc/ssl/certs/${DOMAIN}.crt \
    -subj "/CN=${DOMAIN}"
done

# Verifica que existen:
ls -l /etc/ssl/private | grep www.
ls -l /etc/ssl/certs   | grep www.
```

Crea los vhosts **SSL (443)**:
```bash
# camisetes SSL
tee /etc/apache2/sites-available/www.camisetes.com-ssl.conf > /dev/null <<'APACHE'
<VirtualHost *:443>
    ServerName www.camisetes.com
    ServerAlias camisetes.com
    DocumentRoot /var/www/llista-camisetes
    
    Redirect / https://www.camisetes.com.conf #falta esto

    DirectoryIndex index.html index.php
    ErrorLog  ${APACHE_LOG_DIR}/camisetes_ssl_error.log
    CustomLog ${APACHE_LOG_DIR}/camisetes_ssl_access.log combined

    SSLEngine on
    SSLCertificateFile      /etc/ssl/certs/www.camisetes.com.crt
    SSLCertificateKeyFile   /etc/ssl/private/www.camisetes.com.key

    <Directory /var/www/llista-camisetes>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
APACHE

# llibres SSL
tee /etc/apache2/sites-available/www.llibres.com-ssl.conf > /dev/null <<'APACHE'
<VirtualHost *:443>
    ServerName www.llibres.com
    ServerAlias llibres.com
    DocumentRoot /var/www/llibres-db
    
    Redirect / https://www.llibres.com.conf #falta esto

    DirectoryIndex list.php index.php index.html
    ErrorLog  ${APACHE_LOG_DIR}/llibres_ssl_error.log
    CustomLog ${APACHE_LOG_DIR}/llibres_ssl_access.log combined

    SSLEngine on
    SSLCertificateFile      /etc/ssl/certs/www.llibres.com.crt
    SSLCertificateKeyFile   /etc/ssl/private/www.llibres.com.key

    <Directory /var/www/llibres-db>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
APACHE

# db SSL (si existe phpMyAdmin)
if [ -d /usr/share/phpmyadmin ]; then
tee /etc/apache2/sites-available/www.db.com-ssl.conf > /dev/null <<'APACHE'
<VirtualHost *:443>
    ServerName www.db.com
    ServerAlias db.com
    DocumentRoot /usr/share/phpmyadmin

	Redirect / https://www.db.com.conf #falta esto
	
    DirectoryIndex index.php
    ErrorLog  ${APACHE_LOG_DIR}/db_ssl_error.log
    CustomLog ${APACHE_LOG_DIR}/db_ssl_access.log combined

    SSLEngine on
    SSLCertificateFile      /etc/ssl/certs/www.db.com.crt
    SSLCertificateKeyFile   /etc/ssl/private/www.db.com.key

    <Directory /usr/share/phpmyadmin>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
APACHE
fi

```

>[!warning] FALTA darle permisos!!

>Para que funcione faltaba añadir esto: 
	Redirect / https://www.db.com.conf  # falta esto
   Quizas desde casa haya que incluir el puerto:
	Redirect / https://www.db.com.conf:20780/  # falta esto

Habilita los sitios SSL:

```bash
a2ensite www.camisetes.com-ssl.conf
a2ensite www.llibres.com-ssl.conf
#Miramos si tiene o no phpmyadmin
[ -f /etc/apache2/sites-available/www.db.com-ssl.conf ] && a2ensite www.db.com-ssl.conf 

apachectl configtest
systemctl reload apache2

```

### 5) Resolver los dominios (archivo hosts)

Podriamos mirar primero donde hacemos ping:
```bash
ping ifc.politecnicllevant.cat
```

- Windows: edita `C:\Windows\System32\drivers\etc\hosts` (como admin)
```file
IP desde el instituto:
10.100.145.10 www.camisetes.com camisetes.com
10.100.145.10 www.llibres.com  llibres.com
10.100.145.10 www.db.com       db.com

IP de PROXMOX desde casa:
88.22.131.105 www.camisetes.com camisetes.com
88.22.131.105 www.llibres.com  llibres.com
88.22.131.105 www.db.com       db.com
```

### 6) Comprobacion:
Por proxmox:
```url
https://www.camisetes.com

```


# Alias

Ahora mismo tenemos en: `var/www/llibres-json/images` las imagenes repetidas de `var/www/html/llibres-json/images`, lo que queremos es que las coja del html/llibres-json

Fichero:  `/etc/apache2/sites-available/www.camisetes.com.conf` 
```apache
<VirtualHost *:443>
    ServerName www.camisetes.com
    ServerAlias camisetes.com
    DocumentRoot /var/www/llista-camisetes
    
    Redirect / https://www.camisetes.com.conf
    
    Alias /images /var/www/html/llibres-json/images # Añadimos esto

    DirectoryIndex index.html index.php
    ErrorLog  ${APACHE_LOG_DIR}/camisetes_ssl_error.log
    CustomLog ${APACHE_LOG_DIR}/camisetes_ssl_access.log combined

    SSLEngine on
    SSLCertificateFile      /etc/ssl/certs/www.camisetes.com.crt
    SSLCertificateKeyFile   /etc/ssl/private/www.camisetes.com.key

    <Directory /var/www/llista-camisetes>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>

Igual en las demas
```


