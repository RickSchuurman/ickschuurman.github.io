---
layout: post
title:  "Deploying MEAN app on AWS using Docker!"
date:   2019-04-17 15:15:54
categories: deploy
tags: AWS EC2 Docker Docker-Machine MongoDB NGINX
excerpt: How to deploy a MEAN app on AWS EC2 using Docker
---

* content
{:toc}


This post is a step by step guide on how to deploy your own mean app, consisting of a MongoDB, Node backend and Angular frontend on AWS using Docker.

For this guide we use the MEAN example app which I created for this purpose. You can find it here:

``
https://github.com/RickSchuurman/mean-example-app
``


## Prerequisite

1. Docker installed on your local machine
2. You can access AWS from your terminal using the IAM service


## Launch a new instance on AWS

* Create a EC2 instance with a Docker engine remotely from our computer with Docker Machine.
 
 
    ```bash
     $ docker-machine create 
        --driver amazonec2 
        --amazonec2-access-key <key> 
        --amazonec2-secret-key <key> 
        --amazonec2-region <region> 
        --amazonec2-open-port 27017 
        <name instance>
    ```


  
    > Docker Machine is a tool that lets you install Docker Engine on virtual hosts, and manage the hosts with docker-machine commands. 
      You can use Docker Machine to create Docker hosts on your local machine or on cloud providers like AWS. <br /><br />
      [Link](https://docs.docker.com/machine/overview/) to the Docker Machine homepage.


## Connect Docker Client to the Docker Engine

Connect your Docker Client to the Docker Engine running on AWS EC2 instance


* Setup environment variables

    ``
    $ docker-machine env <name instance>
    ``
    
    ````bash
    export DOCKER_TLS_VERIFY="1"
    export DOCKER_HOST="tcp://**.***.***.**:****"
    export DOCKER_CERT_PATH="../docker-ec2-instance"
    export DOCKER_MACHINE_NAME="docker-ec2-instance"
    # Run this command to configure your shell:
    # eval $(docker-machine env docker-ec2-instance)
    ````
    
* Configure your shell (setup environment variables)
 
    ``
    $ eval $(docker-machine env <name instance>) 
    ``


## Deploy MongoDB on AWS EC2 using Docker

Now we can start with deploying a MongoDb on the just created EC2 instance.


### Create MongoDB container in Docker

``
$ docker run -d -p 27017:27017 --name <container name> -v ~/dataMongo:/data/db mongo
``


  > `docker run -d` <br />
    Run container in background and print container ID
    
  > `-p 27017:27017` <br />
    Configure port, Docker port to published port
    
  > `--name <container name>` <br />
    Assign a name to the container
    
  > `-v ~/dataMongo:/data/db mongo` <br />
    Share container folder with the host `~/Users/..../dataMongo` so data will not be lost if container is deleted. <br />
   


### Create admin user on MongoDB

* Get IP EC2 instances

    ``
    $ docker-machine ip <name instance>
    ``
    
    ````bash
    **.***.***.56
    ````

* Connect MongoDB

    ``
    $ mongo <resulting-ip>
    ``

* Create admin user

    ``
    $ db.createUser(
      {
        user: "<userName>",
        pwd: "<password>",
        roles: [ { role: "userAdminAnyDatabase", db: "admin" } ],
        passwordDigestor : "server"
      }
    )
    ``
   
* Remove container

    ``
    $ docker stop mongodb-mean
    ``
    <br />
    ``
    $ docker rm <container id>
    ``

* Run Mongo with auth enabled

    ``
    $ docker run -d -p 27017:27017 --name <container name> -v ~/dataMongo:/data/db mongo --auth
    ``

* Connect MongoDB with user

    ``
    $ mongo <ip ec2 instance> -u <userName> -p<password>
    ``


### Create user on MongoDB

* Create read/write user on the database mean

    ``
    $ use <database>
    ``

    ``
    $ db.createUser(
        {
          user: "<userName>",
          pwd: "<password>",
          roles: [ { role: "readWrite", db: "<database>" } ],
          passwordDigestor : "server"
        }
      )
    ``


## Deploy Angular frontend using NGINX webserver

For our next step we need to get the frontend running on Docker. We will use NGINX web server to our frontend. 
Docker hub has the required NGINX image, but we will need to add our frontend files to make the complete image.


### Build Angular project

* Set the correct environment setting for the frontend

    ````bash
   pwd: src/environments/environment.prod.ts
    
    export const environment = {
      production: true,
      apiUrl: 'http://<ip ec2 instance>:3000/api'
    };
    ````

* Run Angular build script to create the dist folder

    ``
    $ cd mean-example-app
    ``
    
    ``
    $ npm run build -- --output-path=./dist/out --configuration production
    ``
    
    > The build command creates a new folder called dist for distribution. These are the files we can host on a server and our Angular app will load up. ... Angular will take care of the rest.


### Create NGINX Docker container with Angular frontend

* To create our NGINX image we will use the Dockerfile in the root of our MEAN-example-app:

    ````dockerfile
    FROM nginx
    COPY dist/out /usr/share/nginx/html  
    ````
    
    
    >  A simple Dockerfile can be used to generate a new image that includes the necessary content. <br />
    [Link](https://hub.docker.com/_/nginx) to Docker hub page for more information


* Create Docker image

     ``
    $ cd mean-example-app
    ``
    
    ``
    $ docker build -t mean-nginx .
    ``

* Create Docker container

    ``
    $ docker run -p 80:80 --name nginx-mean -d mean-nginx
    ``
    
    Check if the frontend is running by visiting: `http://ip-given-by-docker-machine`
    

* Check logging Docker container

    If something went wrong you can check the logging
    
    
    ``
    $ docker ps -a
    ``
    
    ``
    $ docker logs mean-nginx
    ``
    
    
## Deploy backend on Docker

For deploying the backend we will follow the same steps as for the frontend. We will start out with the default node-8 image and add our needed files to create a new image. 
Create the following Dockerfile in the backend folder of the mean app:

  ```dockerfile
    FROM node:8
    WORKDIR /app
    
    ENV JWT_KEY=<insert secret key>
    ENV MONGO_PW=mean
    ENV MONGO_URL=<ip adress to mongo>:27017
    
    COPY package.json /app
    RUN npm install
    
    COPY . /app
    
    CMD node server.js
    
    EXPOSE 3000
   ```

Create the image: `docker build -t mean-node .`

Create the container: `docker run -d -p 3000:3000 --name mean-node node-mean`

Check if the backend is running by visiting: `http://<ip-given-by-docker-machine>:3000/api/posts`


## Glossary

| Commands                            Info                                                                                      |
|---------------------------------- |:------------------------------------------------------------------------------------------|
| $ docker info                     |  This command displays system wide information regarding the Docker installation          |
| $ docker active                   |  See which machine is “active”                                                            |
| $ docker logs [container id]      |  Fetch the logs of a container                                                            |
| $ docker ps -a                    |  Show all containers                                                                      |
| $ docker stop [container id]      |  Stop one or more running containers                                                      |
| $ docker rm  [container id]       |  Removes one or more running containers                                                   |
| $ docker rmi  [image name]        |  Removes one or more images                                                               |
| $ docker-machine ssh [instance]   |  Log into remote machine                                                                  |
| $ db.dropUser("[user]")           |  Remove user from MongoDb                                                                 |



