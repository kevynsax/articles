###How to create a Dockerfile for your React Project

The idea here is to use a multiple build step using [node](https://nodejs.org/en/) and [nginx](https://www.nginx.com/) images to build and serve your project

Here are the steps for our project

First we will create a project using [create-react-app](https://github.com/facebook/create-react-app)

```
npx create-react-app my-project
cd my-project
```

To see the running app:
```
npm start
```

Then we can create our Dockerfile

```
//Linux or Mac
touch Dockerfile

//Windows - Powershell
New-Item Dockerfile -ItemType file
```
*yes, dockerfile's do not have any extension or dot in the name*

Copy the following content to you Dockerfile
```dockerfile
FROM node:alpine as build
WORKDIR /app
COPY . .
ENV PATH /app/node_modules/.bin:$PATH
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

####Dockerfile explained

So let's breakdown each part of the dockerfile:

`FROM node:alpine as build`<br/>
We are creating a first intermediate container using node and naming as build

`WORKDIR /app`<br/>
 change the folder that we will apply the following commands for /app
 
 `COPY . .`<br/>
 copying all the files from our project(my-project) to the folder inside the container(/app)
 
 `ENV PATH /app/node_modules/.bin:$PATH`<br/>
 changing the environment path to recognize the scripts inside our node_modules
 
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