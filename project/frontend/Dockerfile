# Téléchargement de l'image officielle node sur base alpine
FROM node:13.12.0-alpine

# Initialiser le répertoire de l'application
WORKDIR /app

# Ajout de la variable d'environnement pour les modules node
ENV PATH /app/node_modules/.bin:$PATH

# Installation des dépendences
COPY package.json .
COPY yarn.lock .

RUN yarn install

# Copie des fichiers du projet
COPY . .

# Commande au lancement du conteneur
CMD [ "yarn", "run", "start" ]