##How to create a Docker for your node project

Our goal in this article is to create one project that uses minimal disk spaces, so we will use node alpine image

First let's create the project<br />
Using terminal(linux and mac) or git bash(windows)
```
mkdir my-project
cd my-project
npm init -y
```

Add on the package.json file the start script
`"start": "node server.js"`

Your package.json file should looks like:
```json
{
  "name": "my-project",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node server.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}

```

Create a file for the server with the name "server.js"

```
touch server.js
```

Write the following code inside the server.js

```javascript
const http = require('http');

const hostname = '127.0.0.1';
const port = 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello World');
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

To see if your project is running fine you can execute

```
npm start
```

Finally we can create a file with the name "Dockerfile"<br/>
*yes, dockerfile's do **not** have any extension or dot in the name*

```
touch Dockerfile
```

inside the Dockerfile use this configurations

```dockerfile
FROM node:alpine
WORKDIR /app
COPY . .
RUN npm install --only=prod
EXPOSE 3000
CMD ["npm", "start"]
```

To test your docker you can run:
```
docker build -t my-project-image .
docker run --name my-project-container -d -p 3000:3000 my-project-image
```

Then you can open your browser na url http://localhost:3000/

###Dockerfile explained
`FROM node:alpine as build`<br/>
We are defining that docker image source will be the node using alpine linux

`WORKDIR /app`<br />
Here we are changing the folder that will be applied the following commands

`COPY . .`<br />
Copying all files from our project's folder to the `/app` folder inside the container

`RUN  npm install --only=prod`<br/>
Installing only the prod dependencies, not install the development dependencies

`EXPOSE 3000`<br />
we making explicit to the docker that he will use the port 3000

`CMD ["npm", "start"]`<br/>
Here we telling the docker which command he has to execute when runs the container   

###Advanced

####Typescript

If you are using typescript you can only copy the compiled files to the new container<br/>
And use a multi step build to minimize the disk usage on your server and optimize the runtime<br />
So your dockerfile will look like:

```dockerfile
FROM node:alpine as build
WORKDIR /app
COPY . .
RUN npm install 
RUN npx tsc

FROM node:alpine
WORKDIR /app
COPY ./package.json ./package.json

RUN npm install --only=prod
COPY --from=build /app/build /app
EXPOSE 3000
CMD ["node", "server"]
```
