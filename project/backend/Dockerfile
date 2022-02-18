# pull official node image
FROM alpine as base

RUN apt install nodejs npm

FROM base as yarn-base
#Yarn packages installer
RUN corepack enable
RUN yarn set version stable

FROM yarn-base
# set working directory
WORKDIR /app

# add `/app/node_modules/.bin` to $PATH
ENV PATH /app/node_modules/.bin:$PATH

# install app dependencies
COPY package.json .
COPY yarn.lock .
RUN yarn install
# RUN npm audit fix

COPY . .

CMD [ "yarn", "run", "start" ]