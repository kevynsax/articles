##Como criar um Docker para o seu projeto Angular

A idéia aqui é criar um multiplo steps build utilizando a imagen do [node](https://nodejs.org/en/) para criar um build optimizado e compacto e [nginx](https://www.nginx.com/) imagem para servir os arquivos compilados.

Aqui estão os passos para criar o container com o nosso projeto Angular dentro  

Primeiro iremos criar um projeto usando o [Angular cli](https://cli.angular.io/): <br />
No terminal(linux ou mac) ou Powershell(windows)

```
ng new --defaults my-project
cd my-project
```

Para ver o projeto rodando você pode executar:
```
ng serve
```

Agora iremos criar o arquivo com o nome "Dockerfile" <br/>
*sim, o Dockerfile não tem extensão ou ponto(.) no nome do arquivo*

```
//Linux or Mac
touch Dockerfile

//Windows - Powershell
New-Item Dockerfile -ItemType file
```

Copie o seguinte conteúdo para o seu Dockerfile

```dockerfile
FROM node:alpine as construcao
WORKDIR /app
COPY . .
RUN npm install --silent
RUN npx ng build --output-path ./dist

FROM nginx:alpine
COPY --from=construcao /app/dist /usr/share/nginx/html
EXPOSE 80
```

Para testar se o seu Dockerfile está correto você pode rodar

```
docker build -t imagem-meu-projeto .
docker run --name container-meu-projeto -d -p 4200:80 imagem-meu-projeto
```

Então poderá abrir o browser na url http://localhost:420 e ver o seu projeto rodando


Para ver as configurações do seu container docker rodando você pode executar:
```
docker ps 
```

####Explicando o Dockerfile

Então vamos explicar parte por parte do dockerfile

`FROM node:alpine as construcao`<br/>
Estamos criando um primeiro container intermediário usando o node e nomeando ele 'construcao'. Este container será descartado quando o build da imagem tiver terminado.

`WORKDIR /app`<br/>
mundando para a pasta '/app' onde iremos aplicar os próximos comandos 
 
`COPY . .`<br/>
copiando todos os arquivos do nosso projeto(meu-projeto) para a pasta dentro do container(/app)
 
`RUN npm install --silent`<br/>
executando o install nas dependências que estão faltando, usando silent para não mostrar nenhum warning
 
`RUN npx ng build --output-path ./dist`<br/>
construindo nosso projeto, nesse step será criado a pasta `./dist`
 
`FROM nginx:alpine`<br/>
Aqui estamos criando o container final, usando o nginx para servir o nosso projeto buildado.
 
`COPY --from=construcao /app/dist /usr/share/nginx/html`<br/>
copiando a pasta build do step de construção(que nós chamamos de contrucao) para pasta padrão do nginx('/usr/share/nginx/html')

`EXPOSE 80`<br/>
Vamos tornar explicito que nosso container irá expor a porta 80


###Avançado

####Rotas

Se você for utilizar **rotas** precisaremos criar um arquivo de configuracão do Nginx, porque o arquivo default não funcionará quando mudarmos as rotas

Aqui estão os passos para criar um arquivo de configuração para o Nginx

Primeiro iremos criar no nosso projeto um arquivo chamado "nginx.conf"

```
//Linux or Mac
touch nginx.conf

//Windows - Powershell
New-Item nginx.conf -ItemType file
```

Dentro do arquivo nginx.conf colocaremos a seguinte configuração 

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

Depois precisaremos muda o Dockerfile para copiar o nosso "nginx.conf" para a pasta "/etc/nginx/"

```
FROM node:alpine as construcao
WORKDIR /app
COPY . .
RUN npm install --silent
RUN npm run build

FROM nginx:alpine
COPY --from=construcao /app/build /usr/share/nginx/html

#A proxima linha é a que copiará o arquivo do nginx para o local correto 
COPY ./nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
```

Para testar se tudo foi configurado corretamente você pode rodar novamente<br/>
*lembre-se de parar e remover o container, para depois roda-lo novamente*
```
docker build -t imagem-meu-projeto .
docker run --name container-meu-projeto -d -p 3000:80 imagem-meu-projeto
```
