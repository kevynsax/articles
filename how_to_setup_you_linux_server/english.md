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
So we going to create a new user on this server that will have permission to clone projects on the folder `/media/sourceCode` and run docker commands.

```
sudo useradd -m -g docker kevyn
```

to use ssh on this user we need to copy the authorized_keys from the home folder to the new user
```
mkdir /home/kevyn/.ssh
cp /root/.ssh/authorized_keys /home/kevyn/.ssh/
sudo chown kevyn /home/kevyn/.ssh/authorized_keys
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
To use a domain to point to your server you will need to create an `a` entry with the value of your server, in this case: `192.81.212.164`<br/>
Using my domain server(godaddy.com) he looks like this
![Domain GoDaddy](https://kevyn.com.br/links/setting-up-server/domain.png)

So you can run your website using http://kevyn.com.br

####HTTPS - SSL
Every web site should be running under https protocol, so one group came together and provided one solution for this.
These organization is called [Let's Encrypt](https://letsencrypt.org) they provide SSL certificates for **free**.

Using root account we are going to install in our server the ssl to respond any request as https

First we need to stop any docker that are using the 80 port
```
docker stop $(docker ps -aq)
docker rm $(docker ps -aq)
```

 