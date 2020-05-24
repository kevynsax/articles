###Como instalar o Docker no Elementary OS

Por que [Elementary OS](https://elementary.io/) não é uma distribuição do Linux mais nova o [Docker](http://www.docker.com/) **não** possui um repositório específico para o Elementary.

Então aqui estão os passos para instalar o Docker no Elementary OS

Primeiro, usando o **terminal** iremos instalar as dependências

```
sudo apt update

sudo apt install \
  apt-transport-https \
  ca-certificates \
  curl \
  gnupg-agent \
  software-properties-common

```

Depois iremos adicionar a chave GPG do Docker e verificar que foi adicionada corretamente:
 
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
```

No terminal deverá aparecer o seguinte código

>pub&nbsp;&nbsp;&nbsp;rsa4096 2017-02-22 [SCEA] <br/>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88 <br/>
>uid&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[ unknown] Docker Release (CE deb) <docker@docker.com> <br />
>sub&nbsp;&nbsp;&nbsp;rsa4096 2017-02-22 [S]


Já que o Docker não possui um repositório iremos utilizar o repositório do Ubuntu(versão Focal)

```
sudo apt-add-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
```

E finalmente podemos instalar o docker e o docker-cli

```
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io
``` 

Se você deseja ter certeza que tudo foi instalado corretamente pode rodar o container Hello World

```
sudo docker run hello-world
```

E deverá ver um output do terminal parecido com esse:

>Unable to find image 'hello-world:latest' locally <br/>
>latest: Pulling from library/hello-world <br />
>0e03bdcc26d7: Pull complete <br />
>Digest: sha256:6a65f928fb91fcfbc963f7aa6d57c8eeb426ad9a20c7ee045538ef34847f44f1 <br />
>Status: Downloaded newer image for hello-world:latest <br />
>
>***Hello from Docker!*** <br />
>This message shows that your installation appears to be working correctly. <br />
>...

####Avançado

Se você quer rodar o docker ***sem  o sudo*** você precisa adicionar o seu usuário no grupo do docker. 


``sudo usermod -aG docker $USER ``

*Lembre-se de ***Deslogar e Logar*** novamente para tornar válida as mudanças*