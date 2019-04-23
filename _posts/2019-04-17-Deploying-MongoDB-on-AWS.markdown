---
layout: post
title:  "Deploying MongoDB on AWS using Docker!"
date:   2019-04-17 15:15:54
categories: deploy
tags: AWS EC2 Docker Docker-Machine MongoDB
excerpt: How to deploy a MongoDB on AWS EC2 using Docker
---

* content
{:toc}

## Prerequisite

1. Docker installed on your local machine
2. You can access AWS from your terminal using the IAM service

##  Launch a new instance on AWS

* Create a EC2 instance with a Docker engine remotely from our computer with Docker Machine.
 
 
    ``
     $ docker-machine create 
        --driver amazonec2 
        --amazonec2-access-key <key> 
        --amazonec2-secret-key <key> 
        --amazonec2-region <region> 
        --amazonec2-open-port 27017 
        <name instance>
    ``


  
    > Docker Machine is a tool that lets you install Docker Engine on virtual hosts, and manage the hosts with docker-machine commands. 
      You can use Docker Machine to create Docker hosts on your local machine or on cloud providers like AWS. <br /><br />
      [Link](https://docs.docker.com/machine/overview/) to the Docker Machine homepage.



## Connect Docker Client to the Docker Engine

Connect your Docker Client to the Docker Engine running on AWS EC2 instance


* Setup environment variables

    ``
    $ docker-machine env <name instance>
    ``
    
    ````
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


## Create MongoDB container in Docker

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
   


## Create admin user on MongoDB

* Get IP EC2 instances

    ``
    $ docker-machine ip <name instance>
    ``
    
    ````
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


## Create user on MongoDB

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



