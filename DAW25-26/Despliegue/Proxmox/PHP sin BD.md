### 1) Instalar Apache, PHP, unzip y Git

> Si **Apache ya está** del apartado anterior, puedes saltarte su instalación. Aun así, dejo todo junto para que sea autocontenida.

```bash
apt update
apt install -y apache2 php libapache2-mod-php unzip git
systemctl enable --now apache2 
```

### 2) Clonar la rama `json` a su DocumentRoot

```bash
# Carpeta destino 
mkdir -p /var/www/html/llibres-json  

# Clonado de SOLO la rama json 
git clone -b json --single-branch https://github.com/antonimoragues/llibres.git /var/www/html/llibres-json
```

### 3) Subir e instalar las imágenes

**Desde mi PC** 
```powershell
scp -i "agonzalez" -P 20722 "/C:\Users\Alexandra\Documents\DAW25-26\DesplegamentWeb\SSH\imatges.zip" root@ifc.politecnicllevant.cat:/var/www/html/

## Si estamos fuera del insti: scp -i "C:\ruta\tu_clave.pem" C:\ruta\local\imatges.zip tuusuario@IP_VM:/home/tuusuario/
```
>Desde el instituto:
>
```bash
# Ajusta ruta local, usuario e IP de tu VM
scp /ruta/local/imatges.zip root@10.100.78.207:/var/www/html/
```

**Dentro de la VM (como root):**

```bash
# Crear carpeta de imágenes si no existe
mkdir -p /var/www/html/llibres-json/images  # Mover el ZIP al destino (ajusta el usuario origen) 
mv imatges.zip /var/www/html/llibres-json/images/  #si estamos en /var/www/html
cd /var/www/html/llibres-json/images 
unzip -q imatges.zip 
rm -f imatges.zip
```


### 4) Propietarios y permisos

```bash
chown -R www-data:www-data /var/www/html/llibres-json 
find /var/www/html/llibres-json -type d -exec chmod 755 {} \; 
find /var/www/html/llibres-json -type f -exec chmod 644 {} \;
```

### 5) Comprobación

Abre en el navegador:

```url
http://ifc.politecnicllevant.cat:20780/llibres-json/list.php
```
