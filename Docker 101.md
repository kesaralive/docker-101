# Docker Compose for absolute begineers 

## Docker installation
RUN below commands in your terminal (Linux, MacOS) or powershell (Windows) to check whether you have installed docker or not.
```
docker version
docker compose version
```
If you haven't install docker in your computer, Please follow the instructions given in the official Docker documentation for the installation.
<a href="https://docs.docker.com/engine/install/" target="_blank">Docker Installation.</a>

## Architecture
Today, we're going to create below docker container stack that contains a frontend container, backend api container and a mysql database container. First we'll start by creating our frontend container.

[![Docker Container Stack](/home/kesaralive/Documents/blog/docker-101/docker-container-stack.png)]

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
> First time setup will be take some time because it has to pull the `httpd` image from the docker image hub. \

At this point, you should be able to see your frontend application at `http://localhost:3000/`. 
