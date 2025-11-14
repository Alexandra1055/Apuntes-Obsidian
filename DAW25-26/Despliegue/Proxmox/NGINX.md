Deshabilitamos Apache para evitar los problemas de los puertos:
(Después veremos como hacerlo en paralelo, ambos encendidos, pero proxmox tiene un limite de puertos)

```bash
systemctl stop apache2
systemctl disable apache2
```

### 1) Clonamos el repositorio 
Repetimos todo lo hecho con apache:

Clonamos

```bash
mkdir -p /var/www/nginx/llista-camisetes
# Clonado de SOLO la rama json 
git clone https://github.com/antonimoragues/llista-camisetes.git

```

Damos permisos
```bash
chown -R www-data:www-data /var/www/nginx/llista-camisetes
find /var/www/nginx/llista-camisetes -type d -exec chmod 755 {} \;
find /var/www/nginx/llista-camisetes -type f -exec chmod 644 {} \;
```

instalamos unzip para descomprimir los archivos 
```bash
apt install -y unzip
```
pasados por scp
```powershell
scp -i "agonzalez" -P 20722 "/C:\Users\Alexandra\Documents\DAW25-26\DesplegamentWeb\SSH\imatges.zip" root@ifc.politecnicllevant.cat:/var/www/nginx/llista-camisetes/images/
```

descomprimimos (nos movemos a la carpeta de images)
```bash
unzip imatges.zip

#reiniciamos apache
systemctl reload apache2
```

### 2) Creamos los server blocks de NGINXS (Virtual Host)
```bash
sudo tee /etc/nginx/sites-availablecamisete.conf >/dev/null <<'APACHE'
server{
	listen 8080;
	listen[::]:8080;
	server_name www.camisetes.com camisetes.com;
	root /var/www/llista-camisetes;
	index index.html index.php
	
	location/{
		try_files $uri $uri/=404;
	}

	location ~/\.ht{
		deny all;
	}
}
APACHE
```

Para poder usar PHP en nginx debemos añadir estas lineas:

```bash
sudo tee /etc/nginx/sites-availablecamisete.conf >/dev/null <<'APACHE'
server{
	listen 8080;
	listen[::]:8080;
	server_name www.camisetes.com camisetes.com;
	root /var/www/llista-camisetes;
	index index.html index.php
	
	location/{
		try_files $uri $uri//index.php?$query_string;
	}

	# Para poder usar PHP en NGINX
	location~\.php${
		include snippets/fastcgi-php.conf;
		fastcgi_pass unix:/run/php/php8.3-fpm.sock;
	}

	location ~/\.ht{
		deny all;
	}
}
APACHE

```

>Lo mismo para llibres y db

### 3)Activar los sites y reiniciar NGINX
```bash
ln -s /etc/nginx/sites-availables/camisetes.conf /etc/nginx/sites-enabled
ln -s /etc/nginx/sites-availables/llibres.conf /etc/nginx/sites-enabled
ln -s /etc/nginx/sites-availables/db.conf /etc/nginx/sites-enabled

nginx -t # Comprobamos
systemctl reload nginx # Reiniciamos
```

### 4) Alias para las imagenes
Cogeremos las imagenes de `/var/www/llibres-db/images`
```bash
cp -a /var/www/llibres-db/images/ /var/www/nginx/llibres-db/images/
```
/comprobar las rutas, no estan bien puestas
### 5)Comprobamos
Entramos en los puertos como antes. 
Comprobamos que estan habilitados en el ~/drivers/etc/host
```notas
88.22.131.105 www.camisetes.com
```

Puerto: 
```http
https://www.camisetes.com:8080
```


# HTTPS en NGINX (como en Apache)
Aprovechamos los de apache 

```bash
ls -l /etc/ssl/private /etc/ssl/certs | grep 'www' # comprobamos
```

Modificamos los ficheros para añadir los OpenSSL certificados:
```bash
sudo tee /etc/nginx/sites-availablecamisete.conf >/dev/null <<'APACHE'
server{
	listen 8080;
	listen[::]:8080;
	server_name www.camisetes.com camisetes.com;
	root /var/www/llista-camisetes;
	index index.html index.php
	
	location/{
		try_files $uri $uri//index.php?$query_string;
	}

	ssl_certificate /etc/ssl/certs/www.camisetes.com.crt
	ssl_certificate_key /etc/ssl/certs/www.camisetes.com.key

	ssl_protocols TLSv1.2 TLSv1.3;
	ssl_prefer_server_chipher on;

	# Para poder usar PHP en NGINX
	location~\php${
		include snippets/fastcgi-php.conf;
		fastcgi_pass unix:/run/php/php8.3-fpm.sock;
	}

	location ~/\.ht{
		deny all;
	}
}
APACHE
```
> Lo mismo en los demas ficheros

Hacemos los enlaces simbolicos
```bash
ln -s /etc/nginx/sites-availables/camisetes_ssl.conf /etc/nginx/sites-enabled
ln -s /etc/nginx/sites-availables/llibres_ssl.conf /etc/nginx/sites-enabled
ln -s /etc/nginx/sites-availables/db_ssl.conf /etc/nginx/sites-enabled

nginx -t # Comprobamos
systemctl reload nginx # Reiniciamos
```
