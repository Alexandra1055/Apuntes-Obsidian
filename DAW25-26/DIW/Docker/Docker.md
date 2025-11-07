---
title: Guía completa de Docker (Node.js, Vite, Sass y Apache)
tags: [docker, node, vite, sass, apache, diw, daw]
---

# Guía completa de Docker (orientada a **Node.js**, **Vite**, **Sass** y **Apache**)

> [!summary] Objetivo
> Entender **qué es cada cosa**, **por qué se usa** y **cómo montar rápido** un entorno **reproducible** para prácticas de **DIW/DAW** con Node.js (npm, Vite), un watcher de Sass y un servidor Apache para producción.

---

## 1) Conceptos clave en 5 minutos

- **Imagen**: plantilla **inmutable** (ej.: `node:22`, `httpd:2.4`).
- **Contenedor**: proceso en ejecución creado a partir de una imagen. **Volátil** por defecto.
- **Dockerfile**: **receta** para construir una imagen propia.
- **`docker run` vs `docker compose`**  
  - `docker run`: lanzar un contenedor con **flags**.  
  - `docker compose`: definir **varios servicios** en `docker-compose.yml` y levantarlos juntos.
- **Bind mount** (volumen de tipo carpeta): montas una **carpeta del host** dentro del contenedor → **cambios en caliente**.
- **Named volume** (volumen con nombre): almacenamiento **gestionado por Docker** (p. ej. para `node_modules`).
- **Red**: los servicios de un mismo compose se **resuelven por nombre** (p. ej. `web` puede llamar a `node-diw`).

### Mapa visual
```mermaid
flowchart LR
  subgraph Host
    A[Tu código<br/>./vite, ./scss, etc.]
    V[(Volumen con nombre<br/>vite_node_modules)]
  end

  A -- bind mount --> C1
  V -- named volume --> C1

  subgraph Compose
    C1[node-diw (Node 22)]
    C2[scss (sass --watch)]
    C3[web (Apache)]
  end

  C1 -- HMR 5173 --> Browser
  C3 -- :80 --> N[Internet]
```

---

## 2) Instalar y comprobar Docker

Instala **Docker Desktop** (Windows/macOS) o **Docker Engine** (Linux).

Comprueba:
```bash
docker --version
docker compose version
```

> [!tip]
> Si no aparece `compose`, usa `docker compose` (v2) y **no** `docker-compose` (v1).

---

## 3) `docker run` – comandos esenciales (con explicación)

**Descargar y ejecutar una imagen (modo demonio):**
```bash
docker run -d --name miweb -p 8080:80 httpd:2.4
```
- `-d`: en segundo plano.  
- `--name miweb`: nombre fácil de recordar.  
- `-p 8080:80`: expone **80 contenedor** en el **8080 host**.  
- `httpd:2.4`: imagen y etiqueta.

**Inspección y control**
```bash
docker ps            # contenedores en ejecución
docker ps -a         # todos (incluye detenidos)
docker logs miweb    # logs del contenedor
docker exec -it miweb bash  # entrar con shell (si la imagen tiene bash/sh)
docker stop miweb    # parar
docker start miweb   # arrancar
docker rm miweb      # borrar (debe estar detenido)
```

**Limpieza**
```bash
docker image ls
docker rmi <imagen>
docker system prune -f  # elimina recursos no usados (¡cuidado!)
```

> [!hint] ¿Cuándo usar `run`?
> **Pruebas rápidas**, un solo contenedor, sin muchas dependencias.

---

## 4) `docker compose` – orquestación simple y legible

**Estructura básica de `docker-compose.yml`**
```yaml
name: "mi-proyecto"
services:
  servicio-ejemplo:
    image: node:22
    tty: true            # asigna TTY (equivalente -t)
    stdin_open: true     # mantiene STDIN abierto (equivalente -i)
    working_dir: /app
    volumes:
      - ./carpeta-local:/app   # bind mount para desarrollo
    ports:
      - "8085:5173"            # host:contenedor
```

**Comandos clave**
```bash
docker compose up -d
docker compose ps
docker compose logs -f web
docker compose exec web sh
docker compose stop
docker compose start
docker compose down          # mantiene volumes
docker compose down -v       # también borra volúmenes (irreversible)
docker compose build --no-cache
```

> [!tip] ¿Cuándo usar compose?
> Siempre que haya **más de un servicio** (Node, Apache, Sass watcher, ngrok, etc.) o necesites un setup **reproducible**.

---

## 5) Dockerfile – construir tus propias imágenes

**Ejemplo: watcher de Sass**

```dockerfile
# Dockerfile-scss
FROM node:18
RUN npm install -g sass
WORKDIR /app
# Entrada por defecto: vigilar carpeta scss y compilar a css
ENTRYPOINT ["sass","--watch","--poll","scss:css"]
```

- `FROM`: imagen base.
- `RUN`: ejecuta comandos **durante la build** (queda en la imagen).
- `WORKDIR`: directorio de trabajo.
- `ENTRYPOINT/CMD`: qué se ejecuta al arrancar el contenedor.

> [!note] `.dockerignore`
> Evita copiar al **build context** carpetas pesadas o secretas: `node_modules`, `.env`, `dist/`, etc.

### Multi-stage (resumen)

**Compilar React/Vite** en una imagen node y **copiar `dist/`** a `httpd` o `nginx` reduce el peso final:

```dockerfile
FROM node:22-alpine AS build
WORKDIR /app
COPY vite/vite-project/package*.json ./
RUN npm ci
COPY vite/vite-project/ .
RUN npm run build

FROM httpd:2.4-alpine
COPY --from=build /app/dist/ /usr/local/apache2/htdocs/app/
```

---

## 6) Node.js + npm (qué hay que saber de verdad)

- `npm init -y` crea `package.json`.

**Dependencias:**
```bash
npm install paquete                 # dependencies
npm install --save-dev paquete      # devDependencies
npm uninstall paquete               # o -g si era global
```

**Scripts en `package.json`:**
```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  }
}
```

- `node_modules` y Git: **no** se commitea → añade `node_modules/` a `.gitignore`.
- Dónde buscar paquetes: <https://www.npmjs.com/>

> [!tip] Truco en Docker
> Usa un **named volume** para `node_modules` y evita choques de SO/permisos.

---

## 7) Vite + HMR en Docker (dev) y Apache (prod)

### Servicio de desarrollo con Vite
```yaml
services:
  vite-dev:
    image: node:22.12
    working_dir: /app
    volumes:
      - ./vite/vite-project:/app
      - vite_node_modules:/app/node_modules  # named volume
    command: sh -c "npm ci && npm run dev -- --host"
    ports:
      - "8085:5173"
    environment:
      - NODE_ENV=development

volumes:
  vite_node_modules:
```

- `--host` permite acceder desde el navegador del **host**.
- Primera vez: `npm ci` instala dependencias **exactas** de `package-lock.json`.

### Construir y servir con Apache (producción)
```yaml
services:
  web:
    build:
      context: .
      dockerfile: Dockerfile-web  # multi-stage: build (node) -> runtime (httpd)
    ports:
      - "85:80"
    environment:
      - NODE_ENV=production
```

**Ejemplo de `Dockerfile-web`** → ver §5.

---

## 8) Watcher de Sass con contenedor dedicado

**`docker-compose.yml`:**
```yaml
services:
  scss:
    build:
      context: .
      dockerfile: Dockerfile-scss
    tty: true
    stdin_open: true
    working_dir: /app
    volumes:
      - ./scss:/app
    # La imagen ejecutará: sass --watch --poll scss:css
```

**Estructura esperada:**
```
./scss/
  scss/   # fuentes .scss
  css/    # salida generada (se crea si no existe)
```

**Alternativa rápida sin construir imagen propia:**
```bash
docker run --rm -it -v "${PWD}":/app -w /app node:18-alpine \
  sh -c "npm install -g sass && sass --watch --poll scss:css"
```

---

## 9) Servicio Node “genérico” para prácticas
```yaml
services:
  node-diw:
    image: node:22.12
    tty: true
    stdin_open: true
    working_dir: /appdiw
    volumes:
      - ./vite:/appdiw
    ports:
      - "8085:5173"
```

- Entra al contenedor: `docker compose exec node-diw sh` y usa `npm create vite@latest`, `npm install`, etc.
- En Linux, si hay permisos: añade `user: "1000:1000"` o `user: node`.

---

## 10) Apache httpd para estáticos o demos
```yaml
services:
  web:
    image: httpd:2.4
    tty: true
    stdin_open: true
    working_dir: /usr/local/apache2/htdocs/
    volumes:
      - ./scss:/usr/local/apache2/htdocs/
    ports:
      - "80:80"
```

Útil para ver **HTML/CSS/JS** sin build, o para publicar el `dist/` de Vite.

> [!hint]
> Si sirves bajo **subcarpetas**, ajusta `base` en `vite.config`.

---

## 11) Servicios extra: **QR (Node)** y **ngrok**

**QR (ejemplo Node)**
```yaml
services:
  qr:
    image: node:22
    tty: true
    stdin_open: true
    working_dir: /app
    volumes:
      - ./qr-app/:/app
    environment:
      - PORT=3004
    ports:
      - "3009:3004"
```

**ngrok (no publiques tu token en Git)**
```yaml
services:
  ngrok:
    image: ngrok/ngrok:latest
    tty: true
    stdin_open: true
    environment:
      - NGROK_AUTHTOKEN=${NGROK_AUTHTOKEN}
    command: http web:80  # túnel al servicio 'web'
```

**`.env`** (junto a `docker-compose.yml`):
```
NGROK_AUTHTOKEN=xxxxxx_tu_token_xxxxxx
```

> [!warning]
> `.env` de **compose** usa `KEY=VALUE` (sin comillas ni `export`). **Añádelo** a `.gitignore`.

---

## 12) Gulp en contenedor (opcional)

**`Dockerfile-gulp`**
```dockerfile
FROM node:22
WORKDIR /app
RUN npm install --global gulp-cli \
 && npm init -y \
 && npm install --save-dev gulp
```

**Servicio**
```yaml
services:
  gulp:
    build:
      context: .
      dockerfile: Dockerfile-gulp
    tty: true
    stdin_open: true
    working_dir: /app
    volumes:
      - ./gulp:/app
```

---

## 13) `.env` y secretos

- **No** subas `.env` a Git; crea **`.env.sample`** con **claves ficticias**.
- Diferencias:
  - `.env` **de compose**: variables usadas en el **YAML**.
  - `.env` **de aplicación**: leído por **tu app** (Vite expone solo prefijo `VITE_` al cliente).

---

## 14) Tareas frecuentes y “por qué”

- Ver contenedores → `docker compose ps` → **¿Está vivo mi servicio?**
- Ver logs → `docker compose logs -f nombre` → errores de arranque, **puertos** ocupados.
- Entrar al contenedor → `docker compose exec nombre sh` → instalar deps, **comandos** locales.
- Reconstruir → `docker compose build` / `--no-cache` → cuando cambias **Dockerfile**.
- Reset duro → `docker compose down -v` → empezar **limpio** (pierdes volúmenes).

---

## 15) Errores típicos (y solución)

- **Puertos en uso** (`bind: address already in use`) → cambia a `"8086:5173"`.
- **HMR no carga desde el host** → asegura `npm run dev -- --host` en `vite-dev`.
- **Permisos (Linux)** → `user: "1000:1000"` o corrige con `chown`.
- **`node_modules` corrupto** → usa **named volume** y ejecuta `npm ci`.
- **`docker exec -it ip bash` no funciona** → usa **nombre** del servicio (`node-diw`) y shell disponible (`sh` en Alpine).
- **Variables no cargan en compose** → `.env` debe estar **junto** al `docker-compose.yml`, formato **KEY=VALUE**.

---

## 16) Plantillas listas para copiar

### A) Desarrollo: **Vite + Sass + Apache (estático)**
```yaml
name: "diw-ut1"
services:
  vite-dev:
    image: node:22.12
    working_dir: /app
    volumes:
      - ./vite/vite-project:/app
      - vite_node_modules:/app/node_modules
    command: sh -c "npm ci && npm run dev -- --host"
    ports:
      - "8085:5173"

  scss:
    build:
      context: .
      dockerfile: Dockerfile-scss
    tty: true
    stdin_open: true
    working_dir: /app
    volumes:
      - ./scss:/app

  web:
    image: httpd:2.4
    working_dir: /usr/local/apache2/htdocs/
    volumes:
      - ./scss:/usr/local/apache2/htdocs/
    ports:
      - "80:80"

volumes:
  vite_node_modules:
```

```dockerfile
# Dockerfile-scss
FROM node:18
RUN npm install -g sass
WORKDIR /app
ENTRYPOINT ["sass","--watch","--poll","scss:css"]
```

### B) Producción: **build de React/Vite** y copia a **Apache**
```yaml
name: "diw-ut1-prod"
services:
  web:
    build:
      context: .
      dockerfile: Dockerfile-web
    ports:
      - "85:80"
    environment:
      - NODE_ENV=production
```

```dockerfile
# Dockerfile-web
FROM node:22-alpine AS react-build
WORKDIR /app
COPY vite/vite-project/package*.json ./
RUN npm ci
COPY vite/vite-project/ .
RUN npm run build

FROM httpd:2.4-alpine
COPY --from=react-build /app/dist/ /usr/local/apache2/htdocs/app/
COPY webpack/demo06/ /usr/local/apache2/htdocs/demo06/
```

---

## 17) Checklist para crear tu propio documento más adelante

- **Objetivo**: ¿dev, prod o ambos?
- **Servicios**: lista (Node dev, Sass, web, API, DB, ngrok…).
- **Puertos**: define `host:contenedor`, evita conflictos.
- **Volúmenes**: **bind mounts** (código) y **named volumes** (`node_modules`).
- **Dockerfiles**: ¿necesitas imagen propia? (watchers, multi-stage).
- **Scripts npm**: `dev`, `build`, `preview` + **README.md**.
- **.env**: variables, `.env.sample`, **no** subir secretos.
- **Comandos**: `up -d`, `logs -f`, `exec`, `down`, `build`.
- **Errores**: anota soluciones para tu equipo/futuro tú.
- **Licencias y versiones**: fija tags (`node:22.12`, `httpd:2.4-alpine`).

---

## 18) Apéndice: tabla rápida de comandos (con explicación)

| Comando | ¿Qué hace? | ¿Cuándo usarlo? |
|---|---|---|
| `docker run -d -p H:C --name N IMG` | Arranca 1 contenedor en segundo plano | Pruebas rápidas |
| `docker ps` / `-a` | Lista contenedores activos / todos | Diagnóstico |
| `docker logs [-f] N` | Muestra logs (seguir con `-f`) | Ver errores/arranque |
| `docker exec -it N sh/bash` | Entra en un contenedor | Depuración/CLI |
| `docker stop/start/rm N` | Parar/arrancar/eliminar | Ciclo de vida |
| `docker compose up -d` | Levanta todos los servicios | Desarrollo diario |
| `docker compose down [-v]` | Baja y elimina (con `-v` borra volúmenes) | Limpieza |
| `docker compose logs -f svc` | Logs de un servicio | Debug continuo |
| `docker compose exec svc sh` | Shell en un servicio | Operativa |
| `docker compose build [--no-cache]` | Reconstruye imágenes | Cambios en Dockerfile |

---

### Notas finales

- Mantén tu repositorio **limpio**: `.gitignore` con `node_modules/`, `dist/`, `.env`, etc.
- Documenta en el **README** cómo arrancar el entorno en **3 comandos**.
- **Fija versiones** de imágenes (`node:22.12`, `httpd:2.4-alpine`) para evitar sorpresas.
- En **Windows PowerShell**, `${PWD}` es la ruta actual; en **Linux/macOS**, usa `$PWD`.
