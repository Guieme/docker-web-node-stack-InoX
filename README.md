# Project Docker

Ce projet à pour objectif de conteneuriser et déployer une application web composée d'une app react en frontend et une API NodeJs en backend, le tout accompagné par une base de donnée MongoDB.

## Dockerfiles

Pour conteneuriser nos applications front et backend, nous utiliser des dockerfile, fichiers dont l'objectif est de décrire les étapes de conteneurisations de l'application. 

```dockerfile
FROM node:13.12.0-alpine
RUN apk update && apk upgrade

WORKDIR /app
ENV PATH /app/node_modules/.bin:$PATH

COPY package.json .
RUN yarn install
COPY . .

CMD [ "yarn", "run", "start" ]
```

Ici on part d'une image de base nodejs sur une distribution alpine, plus légère, largement suffisante pour ce projet.

On y copy l'enssemble du code source de l'application et on vient y ainstaller les dépendences (modules nodes).

Une fois fait, on décrit simplement quel sera le point d'entrée du conteneur, ici il executera simplement `yarn run start`

## Docker-compose

Le fichier docker-compose.yaml est un fichier au format YAML, ou YML, donc l'objectif estde décrire le fonctionnement de nos conteneur et leur comorta$elant dans l'architecture.

```yaml
version: '3'

services:

  frontend:
    build: ./frontend
    container_name: react-app
    ports:
      - 3000:3000

  mongodb:
    image: mongo:latest
    container_name: mongo
    volumes: 
      - 'mongo-data:/data/db'
    networks:
      - backend

  backend:
    build: ./backend
    container_name: nodejs
    environment:
      - MONGO_URI=mongodb://mongo:27017/dbdev
    depends_on:
      - mongodb
    networks:
      - backend
    ports:
      - 8080:8080

volumes:
  mongo-data:

networks:
  backend:
    driver: bridge
```

Ici nous avons défini 3 services :
    
1. L'application react.js, notre frontend, nous spécifions l'emplacement de notre code source et de notre dockerfile, docker se chargera de contruire notre image si elles n'est pas déja existante. Nous effectuerons égalelent un mappage du port 3000 de notre machine hôte vers le port 3000 du conteneur, qui est le point d'entrée de l'application.
    
2.  L'API NodeJs, notre backend, nous spécifions également l'emplacement de notre Dockerfile. Nous utiliserons également une variable d'environnement correspondant à l'adresse de la base de données mongo, afin que l'API puisse communiquer avec la base.
    
3.  La base MongoDB, cette fois-ci directement spécifiée via l'image Docker officielle mongo. Nous crééons également ce que l'on appèle un volume, il permettra à docker de concerver les données de la base, et ce, même si le couteneur vennait à s'arrêter et à redémarer. 

Les servies backend et mongo partagent également un réseau que nous avons appelé "backend", il permet à l'API de communiquer avec la base sans avoir à exposer MongoDB sur un port de la machine et ainsi de protéger un peu mieux l'accès à la base.

## Démarer l'application

Pour démarer l'application rien de plus simple, il suffit de se placer dans le répertoire du projet à l'emplacement du `docker-compose.yaml` et d'executer la commande suivante :
<br>
```
docker-compose up
```