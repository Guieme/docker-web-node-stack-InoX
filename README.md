# Déploiement d'une stack MERN via docker-compose

Ce projet à pour objectif de venir déployer une stack tri-partite composée d'une application frontend React.js, d'une api backend Node.js ainsi qu'une base de donnée NoSQL Mongo DB.
L'application est simple et peu interessante en elle-même, le projet se concentre sur la conteneurisation des services.

---



## DockerFiles

Vous trouverez dans les dépot ./frontend et ./backend deux dockerfiles identiques :

``` Dockerfile
FROM node:13.12.0-alpine

WORKDIR /app
ENV PATH /app/node_modules/.bin:$PATH

COPY package.json .
COPY yarn.lock .

RUN yarn install
COPY . .

CMD [ "yarn", "run", "start" ] 
```

ce Dockerfile permettra de copier ces fichiers dans une image docker, d'installer toutes les dépendences de l'application e de la démarrer au moment du lancement du conteneur.



---

## Docker-compose

à la racine du projet vous trouverez une docker-compose. son rôle est de définir les politiques de lancement de chaque conteneur et les liens qu'ils établissent entre eux.
il défini nos trois services, et pour notre backend et notre front end, vas venir build les Dockerfiles pour utiliser les images docker.

```yml
version: '3'

services:
  mongo:
    image: mongo:5.0.0
    container_name: mongo
    restart: always
    volumes: 
      - 'my-app:/data/db'
    networks:
      - backend

  backend:
    build: ./backend
    container_name: project-backend
    environment:
      - MONGO_URI=mongodb://mongo:27017/dbdev
    depends_on:
      - mongo
    networks:
      - frontend
      - backend

  frontend:
    build: ./frontend
    container_name: project-frontend
    networks:
      - frontend
  
    proxy: 
    image: nginx:latest
    container_name: reverse-proxy
    volumes:
      - ./config/nginx.conf:/etc/nginx/nginx.conf
    ports:
      - 80:80
    networks:
      - frontend
      - backend

volumes:
  my-app:

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
```



---

## Reverse proxy

l'un des conteneur mis en place est sur une image nginx. il agit en tant que proxy inverse, autrement dit, il va permettre de rediriger les flux vers les conteneurs selon le type de requête qu'il recoit en entrée sur le port 80. 

cela permet l'ouverture d'un seul port, à savoir le port 80, et de pouvoir tout de même accéder aux conteneurs backend et front end.

ces rêgles de redirections sont définies dans le fichier ./config/nginx.conf : 

```conf
events{

}
http {
  server {
    server_name localhost;

    location / {
      proxy_pass http://frontend:3000;
    }

    location /api/ {
      proxy_pass http://backend:8080;
      rewrite ^/api(.*)$ $1 break;
    }
  }
}
```
\
\ 
  
  
---

## Architecture projet

l'architecture se contruit donc comme suit :

![Alt text](./project_achitecture.png "a title")




---

## Procédure de lancement

### prerequis

- Docker with linux container installed
- Docker-compose installed

## Build

 - télécharger le dépot git
```
 docker-compose up
```