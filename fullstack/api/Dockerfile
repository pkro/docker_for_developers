# use the node base image from the docker registry
# https://hub.docker.com/_/node
FROM node

# define, create and cd into the working directory inside the container
WORKDIR /app

# copy files from the current directory (where Dockerfile is located)
# to the working directory defined above
COPY package*.json /app/

# install the stuff from package.json inside the container
RUN npm install

# copy everything from the current directory (where Dockerfile is located)
# to the container's working dir
# create a .dockerignore file to exclude directories such as node_modules
COPY . /app/

# open port 4000 of the container
EXPOSE 4000

#run "npm start"
CMD ["npm", "start"]