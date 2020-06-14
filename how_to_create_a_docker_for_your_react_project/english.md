##How to create a Docker for your React Project

The idea here is to use a multiple build step using [node](https://nodejs.org/en/) image to create an optimized build and [nginx](https://www.nginx.com/) image to serve the build of your project

Here are the steps to create our Docker container with our React project

First we will create a project using [create-react-app](https://github.com/facebook/create-react-app)

```
npx create-react-app my-project
cd my-project
```

To see the running app:
```
npm start
```

Then we can create a file with the name "Dockerfile"<br/>
*yes, dockerfile's do **not** have any extension or dot in the name*

```
//Linux or Mac
touch Dockerfile

//Windows - Powershell
New-Item Dockerfile -ItemType file
```


Copy the following content to you Dockerfile
```dockerfile
FROM node:alpine as build
WORKDIR /app
COPY . .
RUN npm install --silent
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
```

To test if your Dockerfile is correct you can run

```
docker build -t my-project-image .
docker run --name my-project-container -d -p 3000:80 my-project-image
```

Then you can open your browser on the link: http://localhost:3000 and see your project running

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
 building our project, in this step will be created the folder `./build`
 
 `FROM nginx:alpine`<br/>
 Here we are creating the final container, using nginx to serve our project
 
 `COPY --from=build /app/build /usr/share/nginx/html`<br/>
 copying the build folder from the node step(that we called build) to the default folder of nginx('/usr/share/nginx/html')

`EXPOSE 80`<br/>
we make explicit that our container will expose the 80 port

###Advanced

####Routes
If you want to use **routes** like [react-router-dom](https://reacttraining.com/react-router/web/guides/quick-start) you will need create a Nginx config file, because the default config file will not work

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
COPY --from=build /app/build /usr/share/nginx/html

#Next line will copy our newly created file to his rightful place 
COPY ./nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
```

To test if all configurations were done right, you can run<br/>
***remember to stop and remove the container before you run again***
```
docker build -t my-project-image .
docker run --name my-project-container -d -p 3000:80 my-project-image
```

####Base Url

Sometimes we want to run our application not on the root of our domain e.g. `http://kevynklava.com/my-project/` for this we going to have change the nginx.conf file and the Dockerfile.

The nginx.conf will look like this:

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

        location /my-project/ {
            root /usr/share/nginx/html;
            try_files $uri /index.html;
        }
    }
} 
```

and we have to tell the React project that he will be running with same base url, to do that we will add an Environment variable on the build stage<br/>
Your Dockerfile will look like:

```dockerfile
FROM node:alpine as build
WORKDIR /app
COPY . .

# Next line will put the base href on index.html as '/still-have-time'
ENV PUBLIC_URL=/my-project

RUN npm install --silent
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html

EXPOSE 80
```

####Base Url + Routes
If you will use routes and have a base url we goint to need do both previous steps and change our routes

So your nginx.conf will look like this:

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

        location /resume/ {
            root /usr/share/nginx/html;
            try_files $uri /index.html;
        }
    }
} 
``` 

Your Dockerfile
```dockerfile
FROM node:alpine as build
WORKDIR /app
COPY . .

# Next line will put the base href on index.html as '/still-have-time'
ENV PUBLIC_URL=/resume

RUN npm install --silent
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html

EXPOSE 80
```

And on your app where you use the `react-router-dom` you will need to change the router
```jsx harmony
<Router basename="/resume">
    <Switch>
        <Route component={HomeComponent} />
    </Switch>
</Router>
```