Entrar a Proxmox:
ssh -i agonzalez root@10.100.78.207
//entramos en el insti

ssh -i agonzalez -p 20722 [root@ifc.politecnicllevant.cat](mailto:root@ifc.politecnicllevant.cat)
//para entrar a proxmos desde cada

Practica:
Dentro da la maquina 
```bash
apt update
apt install -y apache2
systemctl enable --now apache2
# Comprobar que está activo
systemctl status --no-pager apache2
```
## 2) Instalar Git

```bash
apt install -y git
git clone https://github.com/antonimoragues/llista-camisetes.git

```

copiamos lo descargado en var/www/html
```bash
cp llista-camisetes/* /var/www/html/

y damos permisos
chown -R www-data:www-data /var/www/html
find /var/www/html -type d -exec chmod 755 {} \;
find /var/www/html -type f -exec chmod 644 {} \;
```

instalamos unzip para descomprimir los archivos 
```bash
apt install -y unzip
```
pasados por scp
```powershell
scp -i "agonzalez" -P 20722 "/C:\Users\Alexandra\Documents\DAW25-26\DesplegamentWeb\SSH\imatges.zip" root@ifc.politecnicllevant.cat:/var/www/html/images/
```

descomprimimos (nos movemos a la carpeta de images)
```bash
unzip imatges.zip

#reiniciamos apache
systemctl reload apache2
```

