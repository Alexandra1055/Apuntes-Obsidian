_Ngrok_ es un servicio o herramienta que te permite convertir tu servidor local en un servidor accesible mediante un subdominio generado

```yaml
name: "examen-ngrok"
services:
  vite:
    build:
      context: .
      dockerfile: ./vite/Dockerfile
    volumes:
      - ./vite/dist:/output
    tty: true
    stdin_open: true

  nginx:
   build:
    context: .
    dockerfile: ./nginx/Dockerfile
   tty: true
   stdin_open: true
   depends_on:
    - vite
   volumes:
    - ./vite/dist:/usr/share/nginx/html:ro
    - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
   ports:
    - "8080:80"

  ngrok:
    image: ngrok/ngrok:latest
    depends_on:
      - nginx
    environment:
      - NGROK_AUTHTOKEN=358oonw47o1Xs1JFVJR8fuGXdQN_6RSiBunPhNHKeS23MFtdR
    command: http nginx:80
    tty: true
    stdin_open: true
```

>[!warning] en package.json asegurar tener `"dev": "vite --host 0.0.0.0",`


Dockerfile en Vite 
```dockerfile
FROM node:24

WORKDIR /app

COPY vite/package*.json ./
RUN npm install

COPY vite/ .

CMD ["sh", "-c", "npm run build && cp -r dist/* /output"]
```

Dockerfile en nginx:
```dockerfile
FROM nginx:alpine

COPY nginx/default.conf /etc/nginx/conf.d/default.conf
```

Metemos el default.conf
```conf
server {
    listen 80;
    server_name _;

    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri /index.html;
    }
}
```

