# README.md
# Introdução ao Nginx
Este repositório contém informações básicas sobre o Nginx, um servidor web de alta performance e escalabilidade. Aqui você encontrará informações básicas sobre como instalar, configurar e utilizar o Nginx através do Docker Compose.


# Manipulando Nginx via Docker Compose

> Imagem do Nginx via docker-compose:
> 
1. Crie um arquivo `docker-compose.yml` com o seguinte código

```markdown
version: '3.7'
services:
  nginx:
    image: nginx:1.21
    ports:
      - "80:80"
```

1. Suba esse container pelo terminal com o seguinte código

```bash
docker-compose up -d
```

1. Se tudo deu certo, você terá o seguinte container rodando ao usar o comando `docker container ps`

```bash
$ docker container ps
CONTAINER ID   IMAGE        COMMAND                  CREATED         STATUS         PORTS                NAMES
d43a4a21f83c   nginx:1.21   "/docker-entrypoint.…"   4 minutes ago   Up 4 minutes   0.0.0.0:80->80/tcp   nginx-nginx-1
```

<aside>
💡 Até a presente etapa do processo, nós orquestramos um container com a imagem do Nginx via Docker Compose e atribuímos a porta do container à porta da nossa máquina real. Sendo ambas a porta 80!

</aside>

> Nossa próxima etapa é redirecionar o nosso Nginx para que ele mostre uma página HTML criada por nós. Para isso, será necessário a criação de um `volume`
> 

1. No terminal, podemos criar através do comando `docker run` passando a flag `v` e apontando o diretório onde estará o seguinte arquivo html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>My First Nginx Site</title>
  </head>
  <body>
    <header>
      <h1>Hello, world</h1>
    </header>

    <section>
      <p>This is my first ever site using Nginx + Docker to create it!</p>
    </section>

    <footer>
      created by
      <a href="https://www.github.com/pauloricardosb" target="_blank"
        >Paulo Ricardo</a
      >
    </footer>
  </body>
</html>
```

1. Comando para criação do volume

```bash
$ docker run -d --rm --name=nginx -p 80:80 -v $(pwd)/html:/usr/share/nginx/html:ro nginx:1.21
```

Via Docker Compose, basta adicionar a linha `volumes` ao final do arquivo; passando o caminho do nosso volume:

```bash
version: '3.7'
services:
  nginx:
    image: nginx:1.21
    ports:
      - "80:80"
    ***volumes:
      - ./html:/usr/share/nginx/html:ro***
```

> Agora, configuraremos o Nginx para que ele use o SSL. Para isso, precisamos copiar os arquivos de configuração do Nginx
> 

```bash
$ docker run -it --rm --name=nginx -p 446:80 -v $(pwd):/data nginx:1.21 sh

cd /etc
cp -R nginx /data
```

> A partir de agora podemos gerar nossos certificados. Para isso criaremos um diretório e prosseguiremos a partir disso para criá-los
> 

1. Dentro do nosso diretório, usaremos o seguinte comando

```bash
openssl req -newkey rsa:4096 -nodes -sha256 -keyout 192.168.1.172.key -x509 -days 365 -out 192.168.1.172.crt
```

> A saída será um pequeno formulário para preencher pelo terminal como abaixo
> 

```bash
Generating a RSA private key
..............................................................++++
...................................................................................................................................................................++++
writing new private key to '192.168.1.172.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:BR
State or Province Name (full name) [Some-State]:Rio de Janeiro
Locality Name (eg, city) []:Petropolis
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Tmaior
Organizational Unit Name (eg, section) []:Tmaior
Common Name (e.g. server FQDN or YOUR name) []:192.168.1.172
Email Address []: your@email
```

> Com a chave gerada, basta adicionar as informações aos arquivos de configurações!
> 

1. Primeiro, copie os arquivos de configuração para gerar o `ssl.conf`

```bash
cd nginx/conf.d/
sudo cp default.conf ssl.conf
sudo chmod 777 ssl.conf
```

1. No `ssl.conf` adiciona as seguintes linhas que apontarão para os certificados que criamos

```bash
server {
    listen       443 ssl;
    server_name  192.168.1.172;

    ***ssl_certificate      /cert/192.168.1.172.crt;
    ssl_certificate_key  /cert/192.168.1.172.key;***

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
```

1. Feito isso, só precisamos exportar a porta `443` no nosso Docker Compose

```bash
version: '3.7'
services:
  nginx:
    image: nginx:1.21
    ports:
      - "80:80"
			***- "443:443"***
    ***volumes:
      - ./html:/usr/share/nginx/html:ro***
```

<aside>
💡 Pronto! Orquestre seu container e acesse seu site já com seu certificado funcionando!

</aside>

