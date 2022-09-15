# Bootcamp Final Challenge

Soluciones:

## `hello-world-golang` - Golang REST endpoints

Se creo un Dockerfile para poder levantar la aplicacion, especificando el puerto 3002 para que corra la aplicacion.

### DOCKERFILE
```
FROM golang:1.16-alpine
WORKDIR /app
COPY . .
EXPOSE 3002
CMD ["go", "run", "app.go"]
```

## `hello-world-nodejs` - Nodejs REST endpoint

Se modifico el archivo Dockerfile, agregando la linea EXPOSE y modificando la instalacion de las librerias unicamente de typescript, para reducir el tiempo de 
instalacion de las librerias necesarias para la aplicacion. 

### DOCKERFILE
```
FROM node:16
WORKDIR /app
COPY . /app
RUN npm i -D typescript ts-node @types/node @types/express
EXPOSE 3000
CMD ["npm", "run", "start"]
```

## `hello-world-nginx` - Nginx reverse proxy

Se modifico el Dockerfile, agregando la linea EXPOSE faltante. Tambien se modifico el archivo "default.conf", ya que le faltaba las lineas de upstream para 
cada servicio, y las redirecciones para la ejecucion de get-scores e inc-scores.

### DOCKERFILE
```
FROM nginx:alpine
RUN apk add --no-cache jq curl
COPY docker-deps/default.conf /etc/nginx/conf.d/default.conf
EXPOSE 18181
```

### DEFAULT.CONF

```
upstream nodejs_up { 
    server nodejs_service:3000;
}

upstream golang_up {
    server golang_service:3002;
}

server {
    listen 18181;
   
    location /health {
        access_log off;
        log_not_found off;
        return 200 'healthy';
    }

    location /nodejs/hello {
        proxy_pass http://nodejs_up/hello;
        proxy_set_header X-Forwarded-Proto https;
		proxy_set_header X-Url-Scheme $scheme;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header Host $http_host;
		proxy_redirect off;
        
    }

    location /golang/hello {
        proxy_pass http://golang_up/hello;
        proxy_set_header X-Forwarded-Proto https;
		proxy_set_header X-Url-Scheme $scheme;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header Host $http_host;
		proxy_redirect off;
    }
    
    location /golang/get-scores {
        proxy_pass http://golang_up/get-scores;
        proxy_set_header X-Forwarded-Proto https;
		proxy_set_header X-Url-Scheme $scheme;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header Host $http_host;
		proxy_redirect off;
    }

     location /golang/inc-score {
        proxy_pass http://golang_up/inc-score?name=Leonardo;
        proxy_set_header X-Forwarded-Proto https;
		proxy_set_header X-Url-Scheme $scheme;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header Host $http_host;
		proxy_redirect off;
    }
}
```

## `DOCKERCOMPOSE` para levantar los 3 servicios

```
version: "3.8"

services:
  nodejs_service: 
    container_name: nodejs
    image: nodejs
    build: 
      context: ./hello-world-nodejs
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    networks:
      - final

  golang_service:
    container_name: golang
    image: golang
    build: 
      context: ./hello-world-golang
      dockerfile: Dockerfile 
    ports: 
      - "3002:3002"
    networks:
      - final

  nginx_service: 
    container_name: nginx
    image: nginx
    build: 
      context: ./hello-world-nginx
      dockerfile: Dockerfile
    ports:
      - "80:18181"
    depends_on:
      - nodejs_service
      - golang_service
    networks:
      - final

networks:
  final:
    driver: bridge
```
`Servicios`

![servicios](https://github.com/lsmcba/challenge-final/blob/main/assets/Servicios.png)

`Pruebas con CURL`

![prueba](https://github.com/lsmcba/challenge-final/blob/main/assets/Curl.png)

## `WORKFLOW` para GitHub Actions

```
name: challenge_pip

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

env:
  REGISTRY: lsmcba
  IMAGE_NAME_NODEJS: challenge_final_node
  IMAGE_NAME_GOLANG: challenge_final_go
  HEROKU_NAME_NODEJS: challengefinal-dockernode-lsm
  HEROKU_NAME_GOLANG: challengefinal-dockergo-lsm
   
jobs:
  build-docker-nodejs:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Set up QEMU
      uses: docker/setup-qemu-action@v2
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2
    - name: Login to DockerHub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}
    - name: Build and push
      uses: docker/build-push-action@v3
      with:
        context: ./hello-world-nodejs
        push: true
        tags: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME_NODEJS }}:latest

  deploy_heroku_nodejs:
    runs-on: ubuntu-latest
    needs: build-docker-nodejs
    steps:
      - uses: actions/checkout@v3
      - uses: akhileshns/heroku-deploy@v3.12.12 
        with:
          heroku_api_key: ${{secrets.HEROKU_API_KEY}}
          heroku_app_name: ${{env.HEROKU_NAME_NODEJS}}
          heroku_email: "lsmcba@gmail.com"
          appdir: "./hello-world-nodejs"
          usedocker: true
          
  build-docker-golang:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Set up QEMU
      uses: docker/setup-qemu-action@v2
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2
    - name: Login to DockerHub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}
    - name: Build and push
      uses: docker/build-push-action@v3
      with:
        context: ./hello-world-golang
        push: true
        tags: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME_GOLANG }}:latest

  deploy_heroku_golang:
    runs-on: ubuntu-latest
    needs: build-docker-golang
    steps:
      - uses: actions/checkout@v3
      - uses: akhileshns/heroku-deploy@v3.12.12 
        with:
          heroku_api_key: ${{secrets.HEROKU_API_KEY}}
          heroku_app_name: ${{env.HEROKU_NAME_GOLANG}}
          heroku_email: "lsmcba@gmail.com"
          appdir: "./hello-world-golang"
          usedocker: true
```
`GitHub`

![github](https://github.com/lsmcba/challenge-final/blob/main/assets/GitHubActions.png)

`DockerHub`

![dockerhub](https://github.com/lsmcba/challenge-final/blob/main/assets/DockerHub.png)

`Heroku`

![heroku](https://github.com/lsmcba/challenge-final/blob/main/assets/Heroku.png)
          

## Time

`Tiempo de correccion de Golang: 30 Minutos.`

`Tiempo de correccion de nodeJS: 1 Hora.`

`Tiempo de correccion de NGINX: 1 Hora.`

`Tiempo de implementacion de GitHub Action: 1 hora y 30 minutos`
