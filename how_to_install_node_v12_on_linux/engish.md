###How to install Node 12 on Linux

The nodejs repository on the [Debian](https://www.debian.org) based distros still on the old version 8.1<br/>
To install the newest versions we must use node_source to add manually the repository

Here are the steps to install the newest versions

```
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt install -y nodejs
```

Test if the installation was successful

```
node --version && npm --version
```

You should see an output like this:

>v12.16.3 <br/>
>6.14.4

###Advanced
To let the npm verify for newest updates without getting errors like:

>  npm update check failed   

You should change the owner of the configstore folder

```
sudo chown -R $USER:$(id -gn $USER) "/home/$USER/.config/configstore/"
```

###Try out your new environment

Create a folder to a new project

```
mkdir project && cd project
```

Create a file for the server as server.js

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

And then you can run:

```
node server.js
```

If everything is correct you should see on the terminal

>Server running at http://127.0.0.1:3000/

And on the browser(http://127.0.0.1:3000) you will see:

>Hello world