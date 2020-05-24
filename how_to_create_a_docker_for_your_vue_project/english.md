##How to create a docker for your Vue project

Our goal here is minimize disk usage and performance on app usage.
So we will create a multiple build step using the [node](https://nodejs.org/en/) and [nginx](https://www.nginx.com/) images.

Here are the steps to create our Docker container with our Vue project

First let's create a vue project using the [Vue cli](https://cli.vuejs.org/guide/installation.html)

```
vue create my-project -d
cd my-project
``` 

Then we already can create our "Dockerfile"
*yes, dockerfile's do **not** have any extension or dot in the name*

```
//Linux or Mac
touch Dockerfile

//Windows - Powershell
New-Item Dockerfile -ItemType file
```

 Copy the following content to your Dockerfile
 
 ```dockerfile
FROM node:alpine as build
WORKDIR /app
COPY . .
RUN npm install --silent
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist/ /usr/share/nginx/html/
EXPOSE 80
```
 To test if your Dockerfile is correct you can run
 
 ```
 docker build -t my-project-image .
 docker run --name my-project-container -d -p 8080:80 my-project-image
 ```

Then you can open your browser on the link: http://localhost:8080 and see your project running

To see the configs from your running project you can execute
```
docker ps 
```

####Dockerfile explained

So let's breakdown each part of the dockerfile:

`FROM node:alpine as build`<br/>
We are creating a first intermediate container using node and naming as 'build'. This container will be ditched when the build image has finished

`WORKDIR /app`<br/>
change the folder that we will apply the following commands for /app
 
`COPY . .`<br/>
copying all the files from our project(my-project) to the folder inside the container(/app)
 
`RUN npm install --silent`<br/>
running install in our missing dependencies, using silent to not show warnings
 
`RUN npm run build`<br/>
building our project, in this step will be created the folder `./dist`
 
`FROM nginx:alpine`<br/>
Here we are creating the final container, using nginx to serve our project
 
`COPY --from=build /app/dist /usr/share/nginx/html`<br/>
copying the dist folder from the node step(that we called build) to the default folder of nginx('/usr/share/nginx/html')

`EXPOSE 80`<br/>
we make explicit that our container will expose the 80 port

###Advanced

####Routes
If you want to use **routes** you will need create a Nginx config file, if an user access direct the url of some route, nginx will not know how to handle that.

Here are the steps to use a custom Nginx config file


First let's create in our repository the nginx file called "nginx.conf"
```
//Linux or Mac
touch nginx.conf

//Windows - Powershell
New-Item nginx.conf -ItemType file
```

Inside the nginx.conf file we will put this config

```
user nginx;
worker_processes auto;

error_log   /var/log/nginx/error.log warn;
pid         /var/run/nginx.pid;

events {
    worker_connections    1024;
}

http {
    include    /etc/nginx/mime.types;
    server {
        listen       80;
        server_name  localhost;

        location / {
            root /usr/share/nginx/html;
            try_files $uri /index.html;
        }
    }
} 
```

We will need to change our Dockerfile to copy our "nginx.conf" to the "/etc/nginx/nginx.conf" path

```
FROM node:alpine as build
WORKDIR /app
COPY . .
RUN npm install --silent
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html

#Next line will copy our newly created file to his rightful place 
COPY ./nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
```

To test if all configurations were done right, you can run<br/>
>*remember to stop and remove the container before you run again*
```
docker build -t my-project-image .
docker run --name my-project-container -d -p 8080:80 my-project-image
```