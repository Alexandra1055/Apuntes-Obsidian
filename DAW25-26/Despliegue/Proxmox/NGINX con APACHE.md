En proxmox como no somos admin esto no podemos hacerlo

```bash
apt update
apt install -y nginx
# falta comandos para meter mas cosas
```

### 1) Paramos el nginx

Lo paramos para poder darle su puerto

```bash
systemctl stop nginx
systemctl status nginx
```

vamos a /etc/nginx/sites-available/default
En la linea: nano y el directorio
```apache
server {
	listen 8080 default server;
	 listen [ : : ] 8080 defaultserver
		si queremos que convivan habilitamos el puerto 8043
	root /var/www/nginx
	index list.php index.php index.html index.nginx-debian.html
}
```

```bash
mkdir  -p /var/www/nginx

echo "<h1>NGINX Funciona</h1>" | tee /var/www/nginx

nginx -t

systemctl start nginx

systemctl enable nginx


ss -tulnp | grep 

ufw status

```

Habilitamos los puertos:
```bash
ufw allow 8080/tcp
ufw allow 8443/tcp

#Volvemos a comprobra
ufw status
```

A partir de aqui es como los apuntes de NGINX