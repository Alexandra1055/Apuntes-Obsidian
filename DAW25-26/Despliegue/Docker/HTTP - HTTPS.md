---
title: Guía rápida — Virtual Hosts + HTTPS (autosignado)
tags: [apache, vhost, ssl, openssl, docker, phpmyadmin]
---

# 🚀 Guía rápida – Virtual Hosts + HTTPS (autosignado)

## 0) Qué tendrás al final
- `www.camisetes.com` → sirve **`/var/www/llista-camisetes`**
- `www.llibres.com` → sirve **`/var/www/llibres-db`** (con `DirectoryIndex` a `list.php`)
- `www.db.com` → **proxy** a **phpMyAdmin** (contenedor en `127.0.0.1:8083`)
- Todo en **HTTP (80)** y **HTTPS (443)** con **certificados autosignados**.

---

## 1) Copiar contenidos y fijar permisos

```bash
cd /var/www
sudo cp -r html/llista-camisetes/ .
sudo cp -r html/llibres-db/ .

# Propietario correcto (Apache)
sudo chown -R www-data:www-data /var/www/llista-camisetes /var/www/llibres-db

# Permisos: directorios 755, ficheros 644
sudo find /var/www/llista-camisetes -type d -exec chmod 755 {} \;
sudo find /var/www/llista-camisetes -type f -exec chmod 644 {} \;

sudo find /var/www/llibres-db -type d -exec chmod 755 {} \;
sudo find /var/www/llibres-db -type f -exec chmod 644 {} \;
```

---

## 2) VirtualHosts **HTTP (80)**

### 2.1 `www.camisetes.com` (estático)
```bash
sudo tee /etc/apache2/sites-available/www.camisetes.com.conf >/dev/null <<'APACHE'
<VirtualHost *:80>
  ServerName www.camisetes.com
  ServerAlias camisetes.com
  DocumentRoot /var/www/llista-camisetes

  ErrorLog ${APACHE_LOG_DIR}/camisetes_error.log
  CustomLog ${APACHE_LOG_DIR}/camisetes_access.log combined
</VirtualHost>
APACHE
```

### 2.2 `www.llibres.com` (PHP)
```bash
sudo tee /etc/apache2/sites-available/www.llibres.com.conf >/dev/null <<'APACHE'
<VirtualHost *:80>
  ServerName www.llibres.com
  ServerAlias llibres.com
  DocumentRoot /var/www/llibres-db

  # Páginas por defecto
  DirectoryIndex list.php index.php

  ErrorLog ${APACHE_LOG_DIR}/llibres_error.log
  CustomLog ${APACHE_LOG_DIR}/llibres_access.log combined
</VirtualHost>
APACHE
```

> [!alternative] Alternativa con `.htaccess`
> 1) Crea **`/var/www/html/llibres-db/.htaccess`** con:
> ```apache
> DirectoryIndex list.php index.php llist.php llista.php
> Options -Indexes
> ```
> 2) En **`/etc/apache2/apache2.conf`**, dentro de `<Directory /var/www/>`, pon `AllowOverride All`.  
> 3) Activa rewrite si lo usas:
> ```bash
> sudo a2enmod rewrite && sudo systemctl reload apache2
> ```

### 2.3 `www.db.com` (phpMyAdmin)

**Opción A – Proxy al contenedor en `:8083` (recomendada con Docker):**
```bash
sudo a2enmod proxy proxy_http
sudo tee /etc/apache2/sites-available/www.db.com.conf >/dev/null <<'APACHE'
<VirtualHost *:80>
  ServerName www.db.com
  ServerAlias db.com

  ProxyPreserveHost On
  ProxyPass        / http://127.0.0.1:8083/
  ProxyPassReverse / http://127.0.0.1:8083/

  ErrorLog ${APACHE_LOG_DIR}/db_error.log
  CustomLog ${APACHE_LOG_DIR}/db_access.log combined
</VirtualHost>
APACHE
```

**Opción B – `DocumentRoot` local (si instalaste phpMyAdmin en el host):**
```bash
sudo tee /etc/apache2/sites-available/www.db.com.conf >/dev/null <<'APACHE'
<VirtualHost *:80>
  ServerName www.db.com
  ServerAlias db.com
  DocumentRoot /usr/share/phpmyadmin

  ErrorLog ${APACHE_LOG_DIR}/db_error.log
  CustomLog ${APACHE_LOG_DIR}/db_access.log combined
</VirtualHost>
APACHE
```

---

## 3) Habilitar sitios y recargar

```bash
sudo a2ensite www.camisetes.com.conf
sudo a2ensite www.llibres.com.conf
sudo a2ensite www.db.com.conf

sudo apachectl configtest
sudo systemctl reload apache2
```

> [!warning]
> Si hay error, `apachectl configtest` indica el **fichero** y la **línea**.

---

## 4) Resolver nombres en tu PC (`hosts`)

Añade estas líneas apuntando a la **IP** de tu servidor:

- **Linux:** `/etc/hosts`  
- **Windows:** `C:\Windows\System32\drivers\etc\hosts`

```text
10.100.145.10 www.camisetes.com camisetes.com
10.100.145.10 www.llibres.com   llibres.com
10.100.145.10 www.db.com        db.com
```

---

## 5) HTTPS (certificados autosignados con OpenSSL)

### 5.1 Crear claves y certificados (1 año)

```bash
DOMAINS=("www.camisetes.com" "www.llibres.com" "www.db.com")

for DOMAIN in "${DOMAINS[@]}"; do
  sudo openssl req -x509 -nodes -days 365 \
    -newkey rsa:2048 \
    -keyout "/etc/ssl/private/${DOMAIN}.key" \
    -out   "/etc/ssl/certs/${DOMAIN}.crt" \
    -subj "/CN=${DOMAIN}" \
    -sha256
done

# Permisos
sudo chown root:root /etc/ssl/private/*.key /etc/ssl/certs/*.crt
sudo chmod 600 /etc/ssl/private/*.key
sudo chmod 644 /etc/ssl/certs/*.crt
```

### 5.2 Activar SSL y crear VirtualHosts **443**

```bash
sudo a2enmod ssl
```

**`www.camisetes.com-ssl.conf`:**
```bash
sudo tee /etc/apache2/sites-available/www.camisetes.com-ssl.conf >/dev/null <<'APACHE'
<VirtualHost *:443>
  ServerName www.camisetes.com
  ServerAlias camisetes.com
  DocumentRoot /var/www/llista-camisetes

  SSLEngine on
  SSLCertificateFile     /etc/ssl/certs/www.camisetes.com.crt
  SSLCertificateKeyFile  /etc/ssl/private/www.camisetes.com.key

  <Directory /var/www/>
    Options Indexes FollowSymLinks
    Require all granted
    AllowOverride All
  </Directory>

  ErrorLog ${APACHE_LOG_DIR}/camisetes_ssl_error.log
  CustomLog ${APACHE_LOG_DIR}/camisetes_ssl_access.log combined
</VirtualHost>
APACHE
```

**`www.llibres.com-ssl.conf`:**
```bash
sudo tee /etc/apache2/sites-available/www.llibres.com-ssl.conf >/dev/null <<'APACHE'
<VirtualHost *:443>
  ServerName www.llibres.com
  ServerAlias llibres.com
  DocumentRoot /var/www/llibres-db

  SSLEngine on
  SSLCertificateFile     /etc/ssl/certs/www.llibres.com.crt
  SSLCertificateKeyFile  /etc/ssl/private/www.llibres.com.key

  <Directory /var/www/>
    Options Indexes FollowSymLinks
    Require all granted
    AllowOverride All
  </Directory>

  ErrorLog ${APACHE_LOG_DIR}/llibres_ssl_error.log
  CustomLog ${APACHE_LOG_DIR}/llibres_ssl_access.log combined
</VirtualHost>
APACHE
```

**`www.db.com-ssl.conf` (elige A o B):**

**A) Proxy a contenedor `:8083`**
```bash
sudo a2enmod proxy proxy_http
sudo tee /etc/apache2/sites-available/www.db.com-ssl.conf >/dev/null <<'APACHE'
<VirtualHost *:443>
  ServerName www.db.com
  ServerAlias db.com

  SSLEngine on
  SSLCertificateFile     /etc/ssl/certs/www.db.com.crt
  SSLCertificateKeyFile  /etc/ssl/private/www.db.com.key

  ProxyPreserveHost On
  ProxyPass        / http://127.0.0.1:8083/
  ProxyPassReverse / http://127.0.0.1:8083/

  ErrorLog ${APACHE_LOG_DIR}/db_ssl_error.log
  CustomLog ${APACHE_LOG_DIR}/db_ssl_access.log combined
</VirtualHost>
APACHE
```

**B) `DocumentRoot` local phpMyAdmin**
```bash
sudo tee /etc/apache2/sites-available/www.db.com-ssl.conf >/dev/null <<'APACHE'
<VirtualHost *:443>
  ServerName www.db.com
  ServerAlias db.com
  DocumentRoot /usr/share/phpmyadmin

  SSLEngine on
  SSLCertificateFile     /etc/ssl/certs/www.db.com.crt
  SSLCertificateKeyFile  /etc/ssl/private/www.db.com.key

  ErrorLog ${APACHE_LOG_DIR}/db_ssl_error.log
  CustomLog ${APACHE_LOG_DIR}/db_ssl_access.log combined
</VirtualHost>
APACHE
```

### 5.3 Habilitar sitios SSL y recargar

```bash
sudo a2ensite www.camisetes.com-ssl.conf
sudo a2ensite www.llibres.com-ssl.conf
sudo a2ensite www.db.com-ssl.conf

sudo apachectl configtest
sudo systemctl reload apache2
```

> [!note]
> Los navegadores avisarán **“no seguro”** por ser **autosignado** (normal en prácticas).

---

## 6) (Solo si te falta) Lanzar phpMyAdmin con `docker run`

```bash
# Red (si la usas para DB): crea si no existe
docker network create llibres 2>/dev/null || true

# phpMyAdmin publicado en :8083
docker run -d --name phpmyadmin \
  --network llibres \
  -p 8083:80 \
  -e PMA_HOST=db \
  -e PMA_USER=root \
  -e PMA_PASSWORD=secret \
  phpmyadmin/phpmyadmin:latest
```

> Con eso, la opción **Proxy** de `www.db.com` funciona.

---

## 7) Comprobaciones útiles

```bash
# Sitios habilitados
ls -l /etc/apache2/sites-enabled/

# Probar sintaxis
sudo apachectl configtest

# Ver logs si algo no carga
sudo tail -f /var/log/apache2/*error*.log
```
