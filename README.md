# Docker Commands

* docker container ls
* docker container ls -a
* docker container run --name <name of contianer> ngnix 
* docker container run -d <name of contianer> ngnix
* docker container run -it <name of contianer> ngnix sleep 30
* docker container start
* docker container stop
* docker container rm <container_id>
* docker container rm -f <container_id>
* docker container ls -aq
* docker container rm $(docker container ls -aq)
* docker conatiner run -d --name env_test -e "test=123" ngnix
* docker conatiner run -d --name env_test -e "test=123" -e "var2=2345" ngnix
* docker container top
* docker container logs <container_id>
* docker container inspect <container_id>
* docker container run -d -p 8080:80 --name tiger_web ngnix
* docker container exec -it <container_id> /bin/bash
* docker container rename <container_id>  new_name

* docker container kill  <container_id>
* docker container wait <container_id>
* docker container pause <container_id>
* docker container unpause <container_id>
* docker container prune <container_id>
* docker container port <container_id>
* docker container diff <container_id>


* docker container cp abc.txt <container_id>:/rohan/ (Copy file to Container)
* docker conatiner cp <container_id>:/rohan/a.txt .  (Copy file from Container)

* docker container export <container_id> -o my_file.tar
* docker image import my_file.tar my_image_name

* docker conatiner commit <container_id> <new_image_name_to_be_created> (Creating Image from running container)

====================================
## Docker File Instructions:

https://www.fosstechnix.com/dockerfile-instructions/

follow video from Udemy : Ricardo Andre Gonzalez Gomez
=====================================


=====================================
## Docker Container Limit Resource Usage
=====================================
* docker stats container_id  (Check stats of conatiner which shows CPU,memory used by Container)

* docker conatiner run --name ngnix11 -d --memory "200mb" ngnix (Limits Contianer to use provided memory)

* docker conatiner run --name ngnix11 -d --memory "200mb" --cpuset-cpus 0,1 ngnix  (Limits Contianer to use provided CPU)

==========================================
## Dangling Images

These are nothing but orphane images.These are created when create image with same name and same version.

To avoid creating dangling images, always create image with new version.

eg: First time we create 
    
   * docker image build -t rohan:latest .  ( This will create image with image name as "rohan" and tag as "latest")
    
    Now second time if we change something in docker file and recreate image as below,

    * docker image build -t rohan:latest . ( This will recreate image but old image will be shown as null image name and null tag) 


* Finding Dangling images --> docker image ls -f dangling=true -q (here -q gives image id)

* Removing all dangling images --> docker rmi $(docker image ls -f dangling=true -q)


==============================================

## MultiStage Docker File

   Idea of this is to reduce size of final image.

Sample Docker File: https://github.com/ricardoandre97/docker-en-resources/tree/master/docker-images/multi-stage


FROM maven:3.6-alpine as builder


COPY app-maven /app

WORKDIR /app

RUN mvn package

FROM openjdk:8-alpine

COPY --from=builder  /app/target/my-app-1.0-SNAPSHOT.jar /app.jar

CMD java -jar /app.jar


============= Docker Network ==============

Docker has 3 types of Networks,

1. Bridge --> Default Network . This uses bridge driver.

when we create any contianer without Network option then it gets created into bridge n/w

to check it,

docker contianer inspect contianer_name  --> In Network section you will find details

to list Network

docker Network ls

In Default Network all contianers can communicate with each other. But they can only communicate using IP address. they cannot 
use name to communicate . this is drwaback of default network.

eg: docker contianer exec Container1 bash -c "ping 172.0.0.3"  --> allowed
    docker contianer exec Container1 bash -c "ping contianer2"  --> Not allowed

/********* Create own network . *************/
   While creating own network we generally create it using bridge driver.

--> docker network create -d bridge --subnet 172.0.0.2 --gateway 172.0.0.1 my_network

   Now we can assign contianers to ths newly created network,

--> docker contianer run -d -it --name my_cont1 --network my_network ngnix

   Now to communicate within network we can use DNS now,
 
 --> docker contianer exec Container1 -c "ping contianer2"  


 ------ communication of Contianer between 2 different network ----------

 --> docker network connect my_new_net Container2  

 we can also disconnect it 

 docker network disconnect my_new_net Container2

 2. None Network  --> This does not have any driver. so if any contianer is attached to this network it will not have IP, subenet etc.
                     It will just be a contianer without any network.

 3. Host Network  --> Inherit all network properties of your Host machine. Only change is User. Rest eg IP will remain same.
                      But in Bridge network we have different IP's for Contianer and Host

4. Overlay Network  --> 

======================================
## Docker Volume
=======================================

https://training.play-with-docker.com/docker-volumes/


Bind Volumne : In this we create a folder in local machine and map that folder to contianer

docker contianer run --name test -v abc/xyz:/var/lib/mysql mysql:latest

Normal Volumes: This type of volume, docker takes care of it. You just run below commnand and Docker will create volumne for you.

docker volumne create <name>

this volume gets created in docker_root/volumnes/<nameof volume> folder
This can be listed as

--> docker volumne ls

Anonymus volumne:
These are volumes created with out mentioning any mount folder or name as below

docker container run -d --name abc -v /var/lib/www jenkins:latest

Key difference between other volumes and Anonymus volume is,
wehen you delete container with -v flag , anonymus volumne also get deleted but other volumne types reamins persist

docker rm -fv <contianer name>

Sample Volumne in Compose File:
    
    version: '3'
    
services:
    
  web:
    
    container_name: nginx1
    
    ports:
    
      - "8080:80"
    
    volumes:
    
      - "vol2:/usr/share/nginx/html"
    
    image: nginx
    
volumes:
    
  vol2:


=======================================
/********** Init Contianer ********/


/********** Docker Compose ********/

Check Docker Compose files from below Repo

https://github.com/ricardoandre97/docker-en-resources/tree/master/docker-compose



* docker compose up -d

* docker compose -f new_docker_compose_file.yml up -d

* docker compose down

* docker compose build (If we have build image from Docker Compose)

Sample docker-compose.yml file for build

version: '3'

services:
  
  web:
    
    image: my_image
    
    container_name: test_cont
    
    build:
      
      context: build
      
      dockerfile: Dockerfile2


* docker compose logs -f 

--> failure and restart startegry for Contianer
--> ENTRY POINTS in new_docker_compose_file

--> CMD vs ENTRYPOINT
https://www.bmc.com/blogs/docker-cmd-vs-entrypoint/#

