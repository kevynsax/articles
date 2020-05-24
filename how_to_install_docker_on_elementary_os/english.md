###How to install Docker on Elementary OS

Because [Elementary OS](https://elementary.io/) is not a mainstream Linux distribution, [Docker](https://www.docker.com/) **doesn't** have a specific repository for Elementary.

So here are the steps to install Docker on Elementary OS

First let's install the dependencies using the **terminal**

```
sudo apt update

sudo apt install \
  apt-transport-https \
  ca-certificates \
  curl \
  gnupg-agent \
  software-properties-common

```

Then we need to add Docker's GPG key and verify if was the right GPG key:
 
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
```

Then you should see:

>pub&nbsp;&nbsp;&nbsp;rsa4096 2017-02-22 [SCEA] <br/>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88 <br/>
>uid&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[ unknown] Docker Release (CE deb) <docker@docker.com> <br />
>sub&nbsp;&nbsp;&nbsp;rsa4096 2017-02-22 [S]


To add a known repository we will use the Ubuntu Focal Repository 
```
sudo apt-add-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
```

Then we can finally install the docker

```
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io
``` 

If you want make sure that everything is fine you can run Hello World container

```
sudo docker run hello-world
```

And you should see something like:

>Unable to find image 'hello-world:latest' locally <br/>
>latest: Pulling from library/hello-world <br />
>0e03bdcc26d7: Pull complete <br />
>Digest: sha256:6a65f928fb91fcfbc963f7aa6d57c8eeb426ad9a20c7ee045538ef34847f44f1 <br />
>Status: Downloaded newer image for hello-world:latest <br />
>
>***Hello from Docker!*** <br />
>This message shows that your installation appears to be working correctly. <br/>
> ...

####Advanced

If you want to run docker ***without sudo*** you must add your user to the docker's group

``sudo usermod -aG docker $USER ``

*Remember to **Logout and Login** to activate the changes*