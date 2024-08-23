## Containerisation

- it involves building self-sufficient software pakcages that perform consistently, regardless of the machines they run on
- its basically taking the snapshot of a machine, the filesystem and letting you use & deploy it as a construct

## docker

- makes it easier to deploy containers
- allows [container orchestration](## "to put the docker image on cloud machines using services like kubernetes") which makes deployment a breeze
- registry: where images can be deployed on the web

## images vs containers \*

- images behave like a template from which consistent containers can be created. images define initial filesystem of new containers. they bundle app's source code (github only does this) and its dependencies into a self-contained package thats ready to use with a container runtime. within the image, filesystem cotnent is represented as multiple independencies

## dockerfile

```
FROM node:20
# this is called base image below this we can also specify how we want to install node or any stuff mention before

WORKDIR /usr/src/app
# this is where inside the image u want to pull, run ur code, run commands etc

COPY . .
# the first dot defines what needs to be copied in the working directory defined earlier which is in this case is everything except whats mentioned in dockerfile, whereas the second dot represents where inside the working dir the stuff defined earlier needs to be copied which is in this case is directly in the working directory

RUN npm install

EXPOSE 3000
# we need the container to tell that it is ready to accept requests on a specific port where by then the host machine decides which requests should actually be served by the container

CMD ["node", "index.js"]
# used to start the container
```

- in the above yaml code each line till expose excluding it, is a docker layer

- to increase performance what we do is (below) load first the package files, install dependencies then load the index file so that when it changes only the index file layer needs to be rebuilt. with this approach even if there's changes in the dependencies which is rarely the case compared to the index file changes, we can change the package layer & the layers following it :

```
from [whatever]
workdir ["]
copy package* .
RUN npm install
copy . .
other commands
```

similarly for more optimisation we can also use

```
FROM mhart/alpine-node
```

## .dockerignore file

- its just like gitignore. here we mention Dockerfile, node_modules

## docker build

- docker build . -t tag - where . is for current directory

## volumes & networks

- in code to connect backend to db we can use the db container name that we gave while creating it in the following way where mongodb2 is the db container name
  ```
  mongoose.connect("mongodb://mongdb2:27017/mydatabase", {});
  ```
  but as it is not a good idea to hardcode sensitive URLs or info we pass it as env variables with the command like docker run -e var_name=value -e second_var=value ....

## multistage builds

```
FROM node:20 AS base
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install

FROM base AS development
COPY . .
CMD ["npm", "run", "dev"]

FROM base AS production
COPY . .
RUN npm prune --production
CMD ["npm", "run", "start"]
```

this helps in using the image in development and production mode separately by targeting in the docker run command like

```
docker run --target develpoment ...
or --target production ...
```

but when we run the image's container even in dev mode then make changes in the files locally it wont affect the files in the container or trigger something like nodemon as we're changing it in our filesystem not in the container. to solve this what we do is add one more flag to the docker run command like

```
docker run -v .:/usr/src/app ...
```

which asks docker to use the working dir basically like a volume stating that the files in the container and the working dir are basically the same means enabling mutual changes. by mutual changes we mean if we change something in our filesystem it'll propogate in the container and if something changes directly in the container that will also affect files in our filesystem

- now after all this someone who wants to contribute to a project just needs to build it in the dev mode then run it using the -v .:workingDir flag with some other stuff to get started

## docker compose file in YAML

yaml is used for serilisation that is for two diff technologies to agreee on a std for comm

```
services:
  mongodbi_db:
    image: mongo:latest
    ports:
      - "27017:27017"
    volumes:
      - mongodb-data:/data/db
  custom_app:
    build: ./
    ports:
      - "3000:3000"

volumes:
  mongodb-data:
```

## interview questions

- images vs containers

- should u push node modules with src code or pull the node modules inside the image directly? the later as there can be os specific changes in modules

- cmd vs other commands in the dockerfile: cmd is used to get the image that is converted to a container to start running

- y layers? it helps in caching, re-using layers & faster build times

- every container has its own network & one container cant talk to the host machine or to other container by itself which is why we use networks to connect different containers

### y use dbs via docker

- helps to not pollute our filesystem with unnecessary dependencies as containers can be brought down after we're done with them

## commands

- docker run -v volume_name:/db/data -p 27017:27017 mongo
- docker run -p 27017:27017 -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=password --name mongodb --net mongo-network -d mongo
- docker run --network mongo-network -e 'ME_CONFIG_MONGODB_SERVER=mongodb' -e ME_CONFIG_MONGODB_ADMINUSERNAME=admin -e ME_CONFIG_MONGODB_ADMINPASSWORD=password -p 8081:8081 --name mongo-express mongo-express
- docker logs containerId

## assignment

- make docker compose or dockerfile for open source contribution
