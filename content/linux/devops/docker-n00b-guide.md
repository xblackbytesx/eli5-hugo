+++
date = "2017-03-21T16:03:03+01:00"
lastMod = "2018-01-04T10:49:03+01:00"
title = "docker n00b guide"
draft = false

+++

## Installation:
`sudo pacman -S docker docker-compose`

## Add yourself to the docker group:
`sudo gpasswd -a <USER> docker`

WARNING: Every user you add to the `docker` group is root equivalent.  
NOTE: It takes a reboot for this change to take effect.

## Start deamon
`sudo systemctl start docker.service`  
`sudo systemclt enable docker.service`

<!-- # Configuration
Files you need
Dockerfile
.dockerignore
nginx-app.conf
supervisor-app.conf
start.sh
uwsgi.ini
uwsgi_params
ssh dir with private key -->

## Common commands

### Build a container
`docker build -t <docker_name> .`  
`docker run -d -P <docker_name>`

OPTIONAL - Name your own docker instance name:
sudo docker run -d --name <instance_name> -p 81:80 <docker_name>

### Show Docker instances with names
`docker ps`

### Open the bash shell
`docker exec -it <instance_name> bash`

### Stop Docker instance
`docker stop <instance_name>`

<!-- # Publishing -->

<!-- Dockerhub (hosted)
Create an account on Dockerhub

From the root of your project (where your Dockerfile is stored) run these commands
sudo docker login
sudo docker build -t <user>/styleguide-example .
sudo docker push <user>/styleguide-example

On the fresh server now run the following:
docker pull <user>/styleguide-example

Private registry (DIY)
Build your own private registry

Example /start_app_with_docker.sh  
`#! /bin/bash`  

`docker run -d --name styleguide_example --restart=always \`  
`    -e GIT_REPO=<YOUR_GIT_REPO> \`  
`    -e GIT_BRANCH=<YOUR_BRANCH> \`  
`    -e DEBUG=True \`  
`    -e SECRET_KEY='<YOUR_SECRET_KEY>' \`  
`    -p 0.0.0.0:81:80 \`  
`    <user>/styleguide-example` -->

## Removing cruft
`docker rm $(sudo docker ps -a)`  
`docker rmi $(sudo docker images -q)`
