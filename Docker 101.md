# Docker Compose for absolute begineers 

## Docker installation
RUN below commands in your terminal (Linux, MacOS) or powershell (Windows) to check whether you have installed docker or not.
```
docker version
docker compose version
```
If you haven't install docker in your computer, Please follow the instructions given in the official Docker documentation for the installation.
<a href="https://docs.docker.com/engine/install/" target="_blank">Docker Installation.</a>

## A Simple 3-Tier Architecture
Today, we're going to create below simple 3-tier architecture using docker containers. It contains a frontend container, backend api container and a mysql database container. First we'll start by creating our frontend container.

[![Docker Container Stack](/docker-container-stack.png)]

## Frontend App Container
First let's create our main project folder. Inside that folder let's make another folder name `frontend` to manage our frontend application. After that let's add a simple `index.html` file inside the frontend folder to check whether the frontend app is working correctly. \
Next, Let's create a new `docker-compose.yml` file to deploy the frontend app inside a docker container. 

**index.html**
``` html
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>HTML Web Page</title>
  </head>
  <body>
    <h1>Hello, World!</h1>
  </body>
</html>

```
**docker-compose.yml**
``` yml
version: "3"
services:
  frontend:
    image: httpd:latest
    volumes:
      - "./frontend:/usr/local/apache2/htdocs"
    ports:
      - 3000:80
```

### Main Project directory 
After all the settings, you should have a folder structure like this.

    .
    ├── frontend                # Frontend files. (.html/.js/.css)
    │   ├── index.html          # html page 
    └── docker-compose.yml


### What is docker-compose?
Docker compose is a tool that was developed to help define and share multi-container applications. docker-compose reads configuration data from a `YAML` file. You can learn more about docker-compose from <a href="https://docs.docker.com/compose/compose-file/compose-file-v3/" target="_blank">here.</a>

Next, Open up the terminal(Linux, Mac), or powershell(Windows) in your main project directory. and run the command.
```
docker compose up
```
>**Note** \
> You need to have an Internet connection to perform above command for the first time. \
> First time setup will be take some time because it has to pull the `httpd` container image from the <a href="https://hub.docker.com/" target="_blank">docker hub.</a>

At this point, you should be able to see your frontend application at `http://localhost:3000/`. We've used <a href="https://hub.docker.com/_/httpd">`httpd docker container image`</a> to serve this simple frontend app.  

You can press `Ctrl + c` in the terminal to stop all the containers.

### What is Docker Hub?

Docker Hub is the world's largest container image library and community for `container images`. It has over 100,000 __container images__ from various software vendors. And It allows developers to store, manage, and distribute Docker container images. 


## Backend API Container
Now Let's create our backend api container, \
First, Let's add a new folder called `api` inside our main project directory and inside `api` folder let's create a new php file named `index.php`. 

Then let's add a simple `hello, world!` php api endpoint in our __index.php__ file.

**index.php**
``` php
<?php
// Set the response content type to JSON
header('Content-Type: application/json');

// Create a response array
$response = [
    'message' => 'Hello, World!'
];

// Encode the response array as JSON
$jsonResponse = json_encode($response);

// Output the JSON response
echo $jsonResponse;

```

Here we're not going to make the Backend API container directly from a docker image pulled from the docker hub. Instead of that we're going to use a docker Build by making a __Dockerfile__ inside the main project directory.

### What is a Dockerfile?
Dockerfiles are used in Docker to define the configuration and instructions necessary to build a Docker image. Dockerfiles provide a simple and declarative way to automate the creation of Docker images, ensuring consistency and reproducibility across different environments. 

We'll talk more about Dockerfiles in my upcoming articles.
For now, Let's add the below `Dockerfile` inside our main project directory by naming it `Dockerfile` and you don't need to have an file extension for that. 

**Dockerfile**
``` Dockerfile
FROM ubuntu:20.04

LABEL maintainer="hello@kesaralive.com"
LABEL description="Apache / PHP development environment"

ARG DEBIAN_FRONTEND=newt
RUN apt-get update && apt-get install -y lsb-release && apt-get clean all
RUN apt install ca-certificates apt-transport-https software-properties-common -y
RUN add-apt-repository ppa:ondrej/php

RUN apt-get -y update && apt-get install -y \
apache2 \
php8.0 \
libapache2-mod-php8.0 \
php8.0-bcmath \
php8.0-gd \
php8.0-sqlite \
php8.0-mysql \
php8.0-curl \
php8.0-xml \
php8.0-mbstring \
php8.0-zip \
mcrypt \
nano

RUN apt-get install locales
RUN locale-gen fr_FR.UTF-8
RUN locale-gen en_US.UTF-8
RUN locale-gen de_DE.UTF-8

# config PHP
# we want a dev server which shows PHP errors
RUN sed -i -e 's/^error_reporting\s*=.*/error_reporting = E_ALL/' /etc/php/8.0/apache2/php.ini
RUN sed -i -e 's/^display_errors\s*=.*/display_errors = On/' /etc/php/8.0/apache2/php.ini
RUN sed -i -e 's/^zlib.output_compression\s*=.*/zlib.output_compression = Off/' /etc/php/8.0/apache2/php.ini

# to be able to use "nano" with shell on "docker exec -it [CONTAINER ID] bash"
ENV TERM xterm

# Apache conf
# allow .htaccess with RewriteEngine
RUN a2enmod rewrite
# to see live logs we do : docker logs -f [CONTAINER ID]
# without the following line we get "AH00558: apache2: Could not reliably determine the server's fully qualified domain name"
RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf
# autorise .htaccess files
RUN sed -i '/<Directory \/var\/www\/>/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/apache2/apache2.conf

RUN chgrp -R www-data /var/www
RUN find /var/www -type d -exec chmod 775 {} +
RUN find /var/www -type f -exec chmod 664 {} +

EXPOSE 80

# start Apache2 on image start
CMD ["/usr/sbin/apache2ctl","-DFOREGROUND"]

```

After above change, your folder structure should look like this.

    .
    ├── api                     # Backend api files. 
    │   ├── index.php           # php page 
    ├── frontend                # Frontend files. (.html/.js/.css)
    │   ├── index.html          # html page 
    ├── docker-compose.yml
    └── Dockerfile

