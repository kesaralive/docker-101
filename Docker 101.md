# Docker Compose for absolute beginers 

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
First let's create our main project folder. Inside that folder let's make another folder name `frontend` to manage our frontend application. After that let's add a simple `index.html` file inside the frontend folder to check whether the frontend app is working correctly when it's deployed. \
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

Then let's add a simple todos api endpoint in our __index.php__ file.

**index.php**
``` php
<?php
// Set the response content type to JSON
header('Content-Type: application/json');
header('Access-Control-Allow-Origin: http://localhost:3000');
header('Access-Control-Allow-Methods: GET, POST, OPTIONS');

// Create a response array
$response = [
    [
        "_id" => 1,
        "todo" => "Todo 1"
    ],
    [
        "_id" => 2,
        "todo" => "Todo 2"
    ],
    [
        "_id" => 3,
        "todo" => "Todo 3"
    ],
    [
        "_id" => 4,
        "todo" => "Todo 4"
    ]
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

## Let's add a new container to the backend API
Now Let's update the `docker-compose.yml` file with below code.
> You should add the below snippet under the `services` section in your `docker-compose.yml` file. If you have any difficulties, feel free to visit the Github repository to the finalized version.

``` yaml
...
  backend:
    container_name: simple-backend
    build:
      context: ./
      dockerfile: Dockerfile
    volumes:
      - "./api:/var/www/html/"
    ports:
      - 5000:80
```
After adding it, Let's rerun the terminal command `docker compose up` to build and run our `frontend` and `backend` containers.

If everything works well, You should be able to see your simple todos API at http://localhost:5000/ .

## Now Let's call our backend API from frontend.

In order to call our backend API from frontend. We should allow access to our frontend in our backend code. We have already put it in the code. Below line helps frontend to call our simple todo api.

__./api/index.php__
``` php
header('Access-Control-Allow-Origin: http://localhost:3000');
```
>**Note** \
> If you're trying to expose the frontend from a different port instead of port `3000`. Please change the above line in below example. \
> Example: If you're trying to expose it in port `8080`.\
> Update it as below.
> ``` php
> header('Access-Control-Allow-Origin: http://localhost:8080');
> ```

### Let's update our frontend code as below to call the backend API

``` html
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>HTML Web Page</title>
  </head>
  <body>
    <h1>Hello, World!</h1>
    <div id="todos"></div>

    <script>
      fetch("http://localhost:5000/")
        .then((response) => response.json())
        .then((data) => {
          const todos = document.getElementById("todos");

          data?.forEach((item, index) => {
            const todo = document.createElement("h2");
            todo.textContent = item._id + ". " + item.todo;
            todos.appendChild(todo);
          });
        })
        .catch((error) => {
          console.log("Error: ", error);
        });
    </script>
  </body>
</html>
```
Now you can check the result at http://localhost:3000/

Upto this point we've created our frontend container, backend container and called the backend API from the frontend. Next, Let's see how we can add a database container to this.

## Adding the MySQL Database container
First, Let's add the sql dump file that I've created for this tutorial. 
Create a folder named `db` in your main project directory and `dump.sql` file inside it.
Then pase the below code to the `dump.sql` file and save it.

__dump.sql__

``` sql
-- phpMyAdmin SQL Dump
-- version 5.2.1
-- https://www.phpmyadmin.net/
--
-- Host: database:3306
-- Generation Time: Jul 08, 2023 at 05:46 PM
-- Server version: 8.0.33
-- PHP Version: 8.1.17

SET SQL_MODE = "NO_AUTO_VALUE_ON_ZERO";
START TRANSACTION;
SET time_zone = "+00:00";


/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8mb4 */;

--
-- Database: `todo_app`
--

-- --------------------------------------------------------

--
-- Table structure for table `todos`
--

CREATE TABLE `todos` (
  `_id` int NOT NULL,
  `todo` varchar(255) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

--
-- Dumping data for table `todos`
--

INSERT INTO `todos` (`_id`, `todo`) VALUES
(1, 'I will wake up at 4 in the morining.'),
(2, 'I will practice docker for 1 hour.'),
(3, 'I will give time for 2 hours javascript.'),
(4, 'Then I will have breakfast.'),
(5, 'I will give time for 3 hours php.');

--
-- Indexes for dumped tables
--

--
-- Indexes for table `todos`
--
ALTER TABLE `todos`
  ADD PRIMARY KEY (`_id`);

--
-- AUTO_INCREMENT for dumped tables
--

--
-- AUTO_INCREMENT for table `todos`
--
ALTER TABLE `todos`
  MODIFY `_id` int NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=6;
COMMIT;

``` 

After above change, your folder structure should look like this.

    .
    ├── api                     # Backend api files. 
    │   ├── index.php           # php page 
    ├── db                      # SQL dump files. 
    │   ├── dump.sql            # sql dump file 
    ├── frontend                # Frontend files. (.html/.js/.css)
    │   ├── index.html          # html page 
    ├── docker-compose.yml
    └── Dockerfile

Let's append below code snippet to our `docker-compose.yml`. You should append it under `services` section. 

``` yaml
...
  database:
    image: mysql:latest
    environment:
      MYSQL_DATABASE: todo_app
      MYSQL_USER: todo_admin
      MYSQL_PASSWORD: password
      MYSQL_ALLOW_EMPTY_PASSWORD: 1
    volumes:
      - "./db:/docker-entrypoint-initdb.d"
```
Now rerun the terminal command `docker compose up`. 
Then open a new terminal and type `docker ps`. If you've done everything correctly, you should be able to see all three containers running on your docker server. 

## Adding a Browser interface to access the MySQL Database container (PhpMyAdmin)
Next, Let's add a browser interface to access your database.
In this tutorial we use `phpmyadmin` to access our database.
Let's add below code snippet to our `docker-compose.yml` file.

``` yaml
  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    ports:
      - 8080:80
    environment:
      - PMA_HOST=database
      - PMA_PORT=3306
    depends_on:
      - database
```

Now you can access the database container through http://localhost:8080/ by using credentials that we've given when creating our database container.

## Let's retrieve data from our MySql Database container to backend API container


