# Téléchargement de l'image
FROM node:13.12.0-alpine

RUN apk update && apk upgrade

# Initialisation répertoire de l'application
WORKDIR /app

# Ajout variable d'environnement
ENV PATH /app/node_modules/.bin:$PATH

# Installation dépendences
COPY package.json .
RUN yarn install

# Copie fichiers du projet
COPY . .
CMD [ "node", "server.js" ]