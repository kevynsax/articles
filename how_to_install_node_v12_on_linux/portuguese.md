###Como instalar o Node na versão 12

O repositório do nodejs nas distribuições do linux baseados no Debian continuam apontando pra antiga versão 8.1 <br/>
Para instalar as versções mais novas do node é nescessário utilizar o nodeSource para adicionar o repositório manualmente

Aqui estão os passos para instalar as versões mais novas

```
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt install -y nodejs
```

Verifique se a instalação foi feita com sucesso

```
node --version && npm --version
```

Deve aparecer um output parecido com esse:

>v12.16.3 <br/>
>6.14.4

###Avançado
Para permitir que o npm verifique se existe versões mais novas sem ter avisos como esse:
>  npm update check failed   

Você deve mudar o dono da pasta configstore. 

```
sudo chown -R $USER:$(id -gn $USER) "/home/$USER/.config/configstore/"
```

###Faça um teste do seu novo ambiente.

Crie uma pasta para seu novo projeto.
```
mkdir projeto && cd projeto
```

Crie um arquivo para o servidor como servidor.js

```
touch servidor.js
```

Escreva o seguinte codigo dentro do servidor.js

```javascript
const http = require('http');
const host = '127.0.0.1';
const porta = 3000;

const servidor = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Olá mundo');
});

servidor.listen(porta, host, () => {
  console.log(`Servidor está rodando: http://${host}:${porta}/`);
});
```

Então poderá rodar no terminal o comando:

```
node servidor.js
```

Se tudo estiver correto você verá no terminal

>Servidor está rodando: http://127.0.0.1:3000/

E no browser(http://127.0.0.1:3000) você verá:

>Olá mundo