# Docker for developers

## Terms

- Image: contains information needed to build container; basically the binary of the container
- Container: the actual runtime instance of the image; where the ressource / application (MongoDB, React Frontend) is located
- Volume: holds data of containers (database, files etc)
- Networking: virtual network between the containers

## Dockerfile / .dockerignore

Dockerfile: blueprint for container

    # naming conventions
    # current directory = where Dockerfile is located or the directory specified at the end of the docker build command

    # Create a dockerfile for basic node application:

    # use the node base image from the docker registry
    # https://hub.docker.com/_/node
    FROM node

    # define, create and cd into the working directory inside the container
    WORKDIR /user/src/app

    # copy files from the current directory 
    # to the working directory defined above
    COPY package*.json ./

    # install the stuff from package.json inside the container
    RUN npm install

    # copy everything from the current directory
    # to the container's working dir
    # create a .dockerignore file to exclude directories such as node_modules
    COPY . .

    # open port 4000 of the container
    EXPOSE 4000

    #run "npm start"
    CMD ["npm", "start"]


## Create image from from a dockerfile

Create an image with name "pkro/simple-backend" using the current directory:

`docker build -t pkro/simple-backend .`

### Image related commands

- `docker images` shows all images on system
- `docker rmi [image id]` remove an image. ID can be just part of the ID as long as it can be uniquely identified (e.g. 492 instead of 492fb9ae4e7a, just like in git)

## Run container from an image

- Specify the containers port(s) to be published on the host with `-p hostport:containerport` (`-P` to publish all ports)
- use `-d` to detach it and run it in the background; otherwise you can see the stdout of the container in the console
- `-i` makes the stdout/stdin of the container interactive

`docker run -p 4001:4000 pkro/simple-backend`

In case of the example application in basic_node_container, the app serverd inside the container on port 4000 can now be accessed on the host on :

`http://localhost:4001`

### Container related commads

- `docker ps` show all running containers and their image ids
- `docker stop [ID]` what it says
- `docker logs [containerID]` see the stdout output of a container
- `docker push [imagename]` to push image to dockerhub
- `docker pull [imagename]` duh

[Reference of all docker commands](https://docs.docker.com/engine/reference/commandline/docker/)

## Building a backend, fronted and full stack application using docker

![fullstack](readme_images/fullstack.png)

### Practical backend with compose

#### docker-compose

Allows to manage multiple containers in one file.

docker-compose.yml:


    # top level: the service names (defined by user, these could be any)
    app:
    container_name: app
    restart: always
    # build from the Dockerfile in the current directory
    build: .
    # expose these ports, same as with docker run -p hostport:containerport
    ports:
        - "4001:4000"
    # dependencies: this one needs the mongo service defined below
    links:
        - mongo

    mongo:
    container_name: mongo
    # we use the official image from the docker registry,
    # so we don't need a dockerfile / build section for this
    image: mongo
    # expose container port 27017 **to other services**
    # that means this port is NOT accessible by the host machine
    # unless exposed using "ports" (see below)
    expose:
        - '27017'
    # link the data directory in the current folder to 
    # the /data/db directory inside the mongo container
    # so we have a persistent database
    volumes:
        - ./data:/data/db
    # expose port to host machine
    # if only one port is specified, docker will assign a random
    # host machine port to the specified container port
    ports:
        - "27017:27017"


The mongo service can then be accessed by the app container like this (node example), *NOT* by `localhost`:

    mongoose.connect('mongodb://mongo:27017/crm', {
        useMongoClient: true
    });

The containers can then be built with `docker-compose build` or `docker-compose up`, which will build if it doesn't exist and then start the containers.

Single services (containers) can be started using `docker-compose up -d [servicename]`, e.g. `docker-compose up -d mongo`

Use `docker-compose down` or `docker-compose stop` in the docker-compose.yml files directory to stop all containers.

**Exercise: set up the react frontend**