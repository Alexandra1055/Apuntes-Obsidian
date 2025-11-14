
#  GUÍA DETALLADA: INSTALAR DOCKER EN DEBIAN DESDE CERO

##  1. Preparar el sistema

**Comandos:**
```bash
sudo apt update
sudo apt upgrade -y
sudo apt install ca-certificates curl gnupg lsb-release -y
```

**Explicación:**
- `sudo` → ejecuta el comando con **privilegios de administrador** (necesario para configurar Docker y repositorios).
- `apt` → **gestor de paquetes** de Debian/Ubuntu.
- `update` → **actualiza** la lista de paquetes disponibles (no instala).
- `upgrade` → **actualiza** los paquetes instalados a sus versiones más recientes.
- `-y` → **auto-confirma** las preguntas de “¿Desea continuar?”.
- `install` → **instala** paquetes.
- `ca-certificates` → certificados para conexiones **HTTPS** seguras.
- `curl` → herramienta para **descargar** archivos por HTTP/HTTPS.
- `gnupg` → manejo de **claves GPG** (verificar paquetes).
- `lsb-release` → información de la **versión** de Debian instalada.

---

##  2. Añadir el repositorio oficial de Docker

> Docker **no** se instala desde los repositorios de Debian, sino desde los **oficiales de Docker**.

### a) Crear el directorio para las claves GPG
```bash
sudo install -m 0755 -d /etc/apt/keyrings
```
- `install` → crea directorios o copia archivos con **permisos** específicos.
- `-m 0755` → permisos: `7` (propietario `rwx`), `5` (grupo `r-x`), `5` (otros `r-x`).
- `-d` → crea **directorio**.
- **Resultado**: existe `/etc/apt/keyrings` con permisos seguros.

### b) Descargar la clave GPG de Docker
```bash
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
- `curl` → descarga contenido de una **URL**.
  - `-f` **fail silently** (oculta errores HTTP).
  - `-s` **silent** (sin barra de progreso).
  - `-S` **muestra errores** (útil con `-s`).
  - `-L` **sigue redirecciones**.
- `|` (*pipe*) → pasa la salida del 1.º comando al 2.º.
- `gpg --dearmor` → convierte a formato **binario** `.gpg`.
- `-o /ruta/archivo` → **guarda** la salida.
- **Resultado**: clave guardada en `/etc/apt/keyrings/docker.gpg`.

### c) Dar permisos de lectura
```bash
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```
- `chmod` → cambia **permisos**.
- `a+r` → da permiso de **lectura** a todos.

### d) Añadir el repositorio a APT
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
**Desglose:**
- `echo` → imprime texto.
- Línea de repositorio (ejemplo típico):
  ```
  deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian bookworm stable
  ```
- `arch=$(dpkg --print-architecture)` → arquitectura (`amd64`, `arm64`…).
- `signed-by=...` → qué **clave GPG** verifica el repo.
- `$(. /etc/os-release && echo "$VERSION_CODENAME")` → **codename** de Debian (`bullseye`, `bookworm`…).
- `tee archivo` → escribe en el archivo.
- `> /dev/null` → no mostrar salida por pantalla.

---

##  3. Actualizar e instalar Docker
```bash
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

**Componentes:**
- `docker-ce` → **Docker Community Edition** (motor principal).
- `docker-ce-cli` → interfaz **CLI**.
- `containerd.io` → **runtime** que ejecuta contenedores.
- `docker-buildx-plugin` → **mejoras** para `docker build`.
- `docker-compose-plugin` → permite usar `docker compose up` (sin `docker-compose` clásico).

---

##  4. Comprobar que Docker funciona

**Estado del servicio:**
```bash
sudo systemctl status docker
```
- `systemctl` → controla **servicios** (systemd).
- `status docker` → debe salir **active (running)**.

**Habilitar e iniciar:**
```bash
sudo systemctl enable --now docker
```
- `enable` → arranque **automático** al iniciar el sistema.
- `--now` → **inicia** ahora mismo.

---

##  5. Probar con el contenedor “hello-world”
```bash
sudo docker run hello-world
```
- `docker run` → ejecuta un contenedor desde una **imagen**.
- `hello-world` → imagen mínima de **prueba** (si no está, se descarga de Docker Hub).

---

##  6. Usar Docker **sin** `sudo`
```bash
sudo usermod -aG docker $USER
```
- `usermod` → modifica usuarios.
- `-aG docker` → añade al grupo **docker** (sin quitar otros grupos).

**Recargar grupos sin cerrar sesión:**
```bash
newgrp docker
```
- `newgrp` → **recarga** la sesión de grupos.

> [!warning] Importante
> Si tras reiniciar la terminal aún te pide `sudo`, **cierra sesión** y vuelve a entrar o reinicia.

---

##  7. (Opcional) Instalar `docker-compose` clásico
```bash
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

**Explicación:**
- `uname -s` → **SO** (p. ej. `Linux`).
- `uname -m` → **arquitectura** (p. ej. `x86_64`).
- `-L` → sigue **redirecciones**.
- `-o` → archivo de **salida**.
- Se guarda en `/usr/local/bin/` (en el **PATH**).
- `chmod +x` → hace el binario **ejecutable**.
- `docker-compose --version` → verifica instalación.

> [!note]
> Hoy en día se recomienda **`docker compose` (plugin v2)** en lugar del binario clásico `docker-compose`.

---

## 8. Desinstalar Docker

```bash
sudo apt purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```

- `purge` → desinstala y borra **configuración**.
- `rm -rf` → elimina **directorios** de forma recursiva y sin preguntar.  
  Aquí se borran **contenedores, imágenes y volúmenes** almacenados.

> [!danger]
> **Irreversible**: asegúrate de que **no** necesitas esos datos antes de ejecutar.
