##Setting up your linux server

For many projects that we have is important to show to the world.
And one of the ways is to have a computer accessible to anyone.
So in this article we will setup a linux server, point the DNS, create the ssl certificate.

I am going to use Digital Ocean, but there's a lot of options out there where the steps will be about the same.
But right now Digital ocean is the cheaper allocated in US. 

- [Scaleway](https://www.scaleway.com/en/pricing/) - €2.99/month
- [Digital Ocean](https://www.digitalocean.com/pricing/) - U$5/month 
- [Linode](https://www.linode.com/products/nanodes/) - U$5/month
- [AWS](https://calculator.aws/#/estimate?id=6e7652e1811bc1faa27a78ed0ed56e373b9ee584) - U$6.81/month (plus any data transferred)
- [Azure](https://azure.com/e/df92e2fa5e5c48de9f29fdde5afbed9d) - U$13.19/month

After create an account we can create a droplet(server).

We going to use Debian buster and choose the cheapest machine.
![Chosing os and Machine config](https://kevyn.com.br/links/setting-up-server/os-and-price.png)

In this fase you can choose additional disk space, but is not necessary for this example. And choose the location of the server
![Chosing volumes and location](https://kevyn.com.br/links/setting-up-server/volume-and-location.png)

Here you should create a ssh key, is a more security way.
![Chosing ssh key](https://kevyn.com.br/links/setting-up-server/ssh-key.png)

To create a ssh key you can run on an terminal or powershell
```
ssh-keygen
```
and if you let save in the default folder you can get your public key running
```
cat ~/.ssh/id_rsa.pub
```

And then we can add the ssh key on digital ocean 
![Adding our ssh key](https://kevyn.com.br/links/setting-up-server/adding-ssh-key.png)

And finally we can choose the name of the server and finish the creation of our server
![finish create the server](https://kevyn.com.br/links/setting-up-server/finish-creating-server.png)

When the digital ocean finishes to create your server he will define the ip of your server, this is the one that we will use for everything
![ip server](https://kevyn.com.br/links/setting-up-server/ip-our-server.png)

And then we can make our first connection to the server using the terminal or powershell
```
ssh root@192.81.212.164
```
you will see something like<br/>
>The authenticity of host '192.81.212.164 (192.81.212.164)' can't be established.<br/>
ECDSA key fingerprint is SHA256:O4dJ9zjP83haU+cqVfkizsXpFYPZCB7248OhIX+pg78.<br/>
Are you sure you want to continue connecting (yes/no)? yes<br/>
Warning: Permanently added '192.81.212.164' (ECDSA) to the list of known hosts.<br/>
Linux newly-created-server 4.19.0-8-cloud-amd64 #1 SMP Debian 4.19.98-1 (2020-01-26) x86_64<br/>
>
>The programs included with the Debian GNU/Linux system are free software;<br/>
the exact distribution terms for each program are described in the<br/>
individual files in /usr/share/doc/*/copyright.<br/>
>
>Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent<br/>
permitted by applicable law.<br/>
root@newly-created-server:~# <br/>

and then as good practice we should update and upgrade the packages
```
sudo apt update
sudo apt upgrade
```

###Swapfile
This server have not much memory(1gb) so is wise to create a swapfile.
This file will be used by debian to use disk when the system memory is full.

So let's create the swapfile
```
sudo fallocate -l 3g /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
free -h
```
 
you should see some output like<br/>
![status swapfile](https://kevyn.com.br/links/setting-up-server/make-swapfile.png)
 
###Docker
All of my projects I use docker which makes much more easier to migrate my apps into another place. 

So let's install the docker.

First install the dependencies
```
sudo apt update
sudo apt install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```
Then let's add the GPG key
```
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
```
Then we can add the repository
```
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
```
And finally we can install the docker
```
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io
```

Now we can test if the docker is working
```
docker run hello-world
```
If you get an output like this, everything it's ok<br/>
![docker hello world](https://kevyn.com.br/links/setting-up-server/hello-world-docker.png)

To run docker without sudo or using another users, which is true in the server, you will need to add the user to the docker group
```
sudo usermod -aG docker $USER 
```

###User
Is recommended to not use root user for every operation, living only major changes to the root user.
So we going to create a new user that will run docker commands.

```
sudo useradd -m -g docker kevyn
```

to use ssh on this user we need to copy the authorized_keys from the home folder to the new user
```
mkdir /home/kevyn/.ssh
cp /root/.ssh/authorized_keys /home/kevyn/.ssh/
sudo chown -R kevyn /home/kevyn/.ssh
```

now we can disconnect from root user and connect using `kevyn` user
```
exit
ssh kevyn@192.81.212.164
```

###Projects folder

I like to create a folder where I can put all my source code inside.
So let's create a folder and clone my resume project
```
mkdir ~/sourceCode
cd ~/sourceCode
git clone https://github.com/kevynsax/resume.git 
cd resume
```

Then we can build this project and see running on the server
```
docker build -t kevyn-resume .
docker run -d -p 80:80 --name resume kevyn-resume
```
And then you can test opening your browser on the url http://192.81.212.164


###Advanced

####Domain
To use a domain to point to your server you will need to create an `a` entry with the value of your ip server, in this case: `192.81.212.164`<br/>
Using my domain server(godaddy.com) looks like this
![Domain GoDaddy](https://kevyn.com.br/links/setting-up-server/domain.png)

So you can run your website using http://kevyn.com.br <br/>
If want to run your website with `www` remember to add a CNAME with the key www and the value `@` or `.` or `domain.com`

####HTTPS - SSL
Every web site should be running under https protocol, so one group came together and provided one solution for this.
These organization is called [Let's Encrypt](https://letsencrypt.org) they provide SSL certificates for **free**.

Using root account we are going to install in our server the ssl to respond any request as https

First we need to stop any docker that are using the 80 port or 443 port
```
docker stop $(docker ps -aq)
docker rm $(docker ps -aq)
```

We can create a volume to save the nginx and letsencrypt configs
```
docker volume create config
```

Then lets create a docker just to install inside[certbot](https://certbot.eff.org/) which will create the certificate for us
```
docker run --name certbot -p 80:80 -p 443:443 -v config:/etc/nginx -v config:/etc/letsencrypt -it debian bash
``` 

**Inside this container** we will need to install the certbot, so let's do it

First the dependencies
```
apt update
apt install python software-properties-common gnupg 
```
Then lets add the repository
```
add-apt-repository ppa:certbot/certbot
```
**sometimes gives a message of error, on the thread but you can follow the next steps*

```
apt update
apt install python-certbot-nginx
```

Finally we can obtain our certificate
```
certbot --nginx -d example.com -d www.example.com -d example2.com -d www.example2.com
```

You should see some output like this ones:<br/>
![certbot creation](https://kevyn.com.br/links/setting-up-server/create-certbot-first.png)
![certbot creation](https://kevyn.com.br/links/setting-up-server/create-certbot-second.png)

Then we can exit the certbot container and create a nginx container to respond for every request of our server
```
exit
```

####Nginx

To run more than one app in the same server you will have to have an application to manage all requests.<br />
Here I decided to use nginx he will redirect all requests.

First we will create a network to put all container that will need to be exposed to outside world
```
docker network create br0
```

And re-run the resume container to expose him under https protocol
```
docker rm resume
docker -d --name resume --network br0 kevyn-resume
``` 
ps: we don't need to expose his port since that nginx will do that for us.

So lets create a Nginx docker using the already created config

***Important: this configs must be done by the `root` user***
```
docker run --name nginx --network br0 -p 80:80 -p 443:443 -v config:/etc/nginx -v config:/etc/letsencrypt -it nginx bash 
```

we will need to edit the file `/etc/nginx/sites-enabled/default`
```
apt update 
apt install nano
nano /etc/nginx/sites-enabled/default
```

and the file: `/etc/nginx/sites-enabled/default` should look something like this:
```
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    
    return 301 https://$host$request_uri;
}

server {
    listen [::]:443 ssl ipv6only=on;
    listen 443 ssl;
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    location / {
        proxy_revert http://resume/;
    }
}
```
where `example.com` should be your domain

then we can reload nginx config
```
nginx -s reload
```
Now you can test your website and should redirect to the https version

####SSH
Disable root login. You should always avoid use root login so you should disable it. And for any reason you want to use the root login you can login into Digital Ocean dashboard and open a console to use the root login inside their platform 

Edit the file of ssh config and disable root login
```
nano /etc/ssh/sshd_config
```

change the line that enable root login
```
PermitRootLogin no
```

###Conclusion

This is the steps that I take to setup my server. So if you have any thing to add I will be glad to know. 