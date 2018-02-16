# Infra

## Domínio

Registro.br - lucasmarques73.com.br  
R$ 40,00 /ano.

## Servidor VPS Locaweb

Foi contrato um plano básico da locaweb.
Servidor disponível.
> 1vCPU  
> 512 MB  
> 20 GB SSD  
> 1 TB de Transferência
> SO Ubuntu 16.04  

R$ 17,90 /mês

### Configurando DNS

Adicionado no site registro.br os DNS da locaweb.  
Adicionado o domínio no painel administrativo da locaweb.  
Na Locaweb foi necessário adicionar Zonas de DNS.  

```
Entrada : www
Tipo    : CNAME
Conteúdo: lucasmarques73.com.br
------------------------------------------------
Entrada : ns1
Tipo    : CNAME
Conteúdo: ns1.locaweb.com.br
------------------------------------------------
Entrada : ns2
Tipo    : CNAME
Conteúdo: ns2.locaweb.com.br
------------------------------------------------
Entrada : ns3
Tipo    : CNAME
Conteúdo: ns3.locaweb.com.br
------------------------------------------------
Entrada : .
Tipo    : A
Conteúdo: #Ip do servidor
```

Após isso foi só esperar propagar o DNS.
Acompanhei pelo site https://www.whatsmydns.net/

### Acessando o servidor

É concedido acesso root ao servidor, o acesso é feito via ssh e pode ser via chave ssh ou senha.  
É definido no momento em que criamos o nosso servidor. Optei por senha.  

```
ssh root@lucasmarques73.com.br
```
Após acesso via root, primeira etapa foi criar um novo usuário e conceder privilégios de root para o mesmo.

```
adduser lucas

usermod -aG sudo lucas

usermod -aG root lucas
```

Durante a criação do usuário foi pedido para responder algumas perguntas, começando pela senha e alguns dados pessoais. Eu preenchi somente a senha e o nome completo.  
Adicionei ao grupo `sudo` seguindo um tutorial da Digital Ocean mas vi o grupo do usuário `root` e era outro chamado também de `root`, então resolvi adicionar meu usuário aos dois grupos.

### Removendo acesso SSH do usuário root

Após criar o meu usuário pessoal eu não queria que fosse possivel conectar diretamente pelo usuário `root`.  
Então eu desabilitei nas configurações do ssh.  

* Todas as alterações a seguir devem ser feitas por um usuário com privilégios de `root`

```
sudo su
```

Editando arquigo de configuração do ssh:

```
nano /etc/ssh/sshd_config
```

Definimos o paramêtro `PermitRootLogin` como `no`;

E após isso, reiniciamos o serviço ssh:

```
service sshd restart
```

Agora não conseguimos mais conectar via ssh com o usuário root.

### Servidor Web

Optei pelo NGINX, mas antes de instalar ele precisamos instalar o pacote `build-essential`.  
```
sudo apt install build-essential
```

Foi instalado o NGINX para ser o servidor web. 

```
sudo apt install nginx
```

### Configuração de VHost e subdomínios

Configurei dois vhosts para testes, usando um como um subdominio.  
As configurações são básicamente as mesmas diferenciando dois paramêtros.

Alterando para usuário `root`

```
sudo su
```

Criando o arquivo.

```
touch /etc/nginx/sites-available/homepage.conf
```

Editando o arquivo.

```
nano /etc/nginx/sites-available/homepage.conf
```
```
server{
	server_name www.lucasmarques73.com.br lucasmarques73.com.br;
	root /home/lucas/homepage;
	
	index index.html;

	charset utf-8;

	location / {
		try_files $uri $uri/ =404;
	}

}
```

* Lembrando que temos que ter dentro de `/home/lucas/homepage` um arquivo com o nome de `index.html` que será nossa pagina inicial.

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to lucasmarques73.com.br</h1>
</body>
</html>
```

Após criar ele eu o ativei.

```
ln -s /etc/nginx/sites-available/homepage.conf /etc/nginx/sites-enabled/homepage.conf
```

E removi o `default` criado pelo nginx.

```
rm /etc/nginx/sites-enabled/default
```

---
Para o subdomínio o procedimento foi básicamente o mesmo.  
Criando um novo projeto.
```
mkdir /var/www/teste

touch /var/www/teste/index.html

nano /vat/www/teste/index.html
```
Nova página.

```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to teste.lucasmarques73.com.br</h1>
</body>
</html>
```
Novo vhost.

```
server{
	server_name teste.lucasmarques73.com.br www.teste.lucasmarques73.com.br;
	root /var/www/teste;
	
	index index.html;

	charset utf-8;

	location / {
		try_files $uri $uri/ =404;
	}

}
```

Além disso, na locaweb adicionei zonas de dns.

```
Entrada : teste
Tipo    : CNAME
Conteúdo: lucasmarques73.com.br
------------------------------------------------
Entrada : teste
Tipo    : A
Conteúdo: #Ip do servidor
```

### Configurando SSL

Decidi por curiosidade instalar certificados SSL no meu servidor usando o [letsencrypt](https://letsencrypt.org/) por ser gratuito.

Segui o tutorial da [SegInfo](https://seginfo.com.br/2015/12/09/guia-passo-a-passo-para-instalar-o-certificado-ssl-gratuito-da-lets-encrypt-no-seu-site-2/)

É necessário clonar o projeto do github então o primeiro passo foi instalar o git.

```
apt install git
```

Clonando o projeto

```
git clone https://github.com/letsencrypt/letsencrypt

cd letsencrypt

./letsencrypt-auto
```
Neste momento eu tive um problema e fui ao google.

```
setuptools pkg_resources pip wheel failed with error code 1
```

Solução encontrada através da [Issue](https://github.com/certbot/certbot/issues/2883)

```
apt install letsencrypt

export LC_ALL="en_US.UTF-8"

export LC_CTYPE="en_US.UTF-8"

```

Após o procedimento anterior rodei novamente o `letsencrypt-auto`, ele foi o responsável de fazer toda a configuração nos hosts.

Dentro da pasta `letsencrypt`
```
./letsencrypt-auto
```

Ele fez algumas perguntas pedindo email, confirmação dos termos.  
Após isso ele mostrou meus hosts e foi só selecionar qual eu queria colocar o certificado e depois optei pela ferramenta sobrescrever meu arquivo de configuração do host. 
Com isso já consegui acessar meu domínio pela conexão segura `https`.  

Como resultado meu arquivo de configuração do host ficou da seguinte maneira. 

 ```
 server{
	server_name www.lucasmarques73.com.br lucasmarques73.com.br;
	root /home/lucas/homepage;
	
	index index.html;

	charset utf-8;

	location / {
		try_files $uri $uri/ =404;
	}
 # managed by Certbot

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/www.lucasmarques73.com.br/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/www.lucasmarques73.com.br/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot


}
server{
    if ($host = www.lucasmarques73.com.br) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    if ($host = lucasmarques73.com.br) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


	server_name www.lucasmarques73.com.br lucasmarques73.com.br;
    listen 80;
    return 404; # managed by Certbot

}
```


## Git Hooks

Configurei um hook no git para sempre que fizer o push para meu servidor ele já faça o pull e atualize meu repositório.

Criando repositório responsável pelo hook
```
mkdir homepage.git

cd homepage.git

git init --bare
```

repositório do projeto
```
cd /home/lucas

git clone homepage.git
```

No final vamos ter duas pastas
```
homepage.git #Responsável pelo hook

homepage    #Responsável pelos arquivos do projeto
```

Criando o hook
```
touch /home/lucas/homepage.git/hooks/post-receive
```

Editando o arquivo do hook
```
nano /home/lucas/homepage.git/hooks/post-receive
```
```
unset $(git rev-parse --local-env-vars)
cd /home/lucas/homepage
git pull
```

Alterar a permissão do arquivo
```
chmod +x homepage.git/hooks/post-receive
```
O hook `post-receive` vai executar o script dentro do arquivo depois que todo o processo esteja concluído.  
Entrando dentro da pasta do projeto e dando um `pull` atualizando o projeto.

Em nosso computador local, devemos clonar o projeto do nosso servidor remoto para trabalharmos nele.
```
git clone lucas@lucasmarques73.com.br:/home/lucas/homepage.git
```

Adicione ou caso já tenha um arquivo, edite ele para testarmos o hook.
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>Welcome to lucasmarques73!</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <style>
        body {
            width: 35em;
            margin: 0 auto;
            font-family: Tahoma, Verdana, Arial, sans-serif;
        }
    </style>    
</head>
<body>
    <h1>Welcome to lucasmarques73.com.br</h1>
    <p>Works Hooks</p>
    <p>Works Hooks</p>
</body>
</html>
```

Após isso vamos subir essa alteração e automáticamente ela já deve ser refletida no seu site em produção.
```
git add .

git commit -m "testing hooks"

git push origin master
```

Além de subir direto para produção, utilizei o github.com para guardar o repositório.
```
git remote add github git@github.com:lucasmarques73/homepage.git

git push github master
```

Com isso tenho meu projeto no meu servidor e também no meu github e toda vez que subir uma alteração, tenho que subir para os dois repos remotos.


## Instalar NVM e NodeJS

Como minha ideia é fazer alguns projetos usando NodeJS, resolvi instalar o nvm para ter um melhor gerenciamento sobre a versão do meu Node.

Antes de começar com o NVM tenho que instalar alguns pacotes.
```
sudo apt update
sudo apt install build-essential libssl-dev
```

Baixando NVM
```
curl -sL https://raw.githubusercontent.com/creationix/nvm/v0.33.8/install.sh -o install_nvm.sh
```
* Lembrando que estou baixando a versão v0.33.8 e pode sofrer alterações.

Executando o script
```
bash install_nvm.sh
```
Ele irá instalar o software dentro de um subdiretório do seu diretório home em `~/.nvm.` Ele irá adicionar também as linhas necessárias ao seu arquivo `~/.profile` para utilizar o arquivo.  

Para obter acesso à funcionalidade do nvm, você precisará sair e se logar novamente, ou você pode varrer o arquivo `~/.profile` de modo que sua sessão atual saiba sobre as alterações:

```
source ~/.profile
```
* Texto retirado do tutorial da DigitalOcean.

Listando as versões disponíveis do Node:
```
nvm ls-remote
```

Instalando a versão:
```
nvm install v9.5.0
```

Testando o Noje e o Npm:
```
node --version #v9.5.0
npm --version #5.6.0
```


## Fontes

* https://github.com/certbot/certbot/issues/2883
* https://seginfo.com.br/2017/06/29/diga-adeus-ao-ssl-e-versoes-antigas-do-tls/
* https://showmethecode.com.br/2017/02/07/blog/
* http://adrianorosa.com/blog/nginx/setup-server-block-virtual-host-com-nginx.html
* https://gist.github.com/vedovelli/a50fdd9c9b745b61407a
* https://blog.fibrasites.com.br/transformando-subdominios-em-rotas-com-o-nginx/
* https://gist.github.com/ianjuma/9009490
* https://wiki.locaweb.com.br/pt-br/Redirecionamento_via_zona_de_DNS
* https://tchubirabiron.wordpress.com/2012/05/14/como-desabilitar-o-login-do-root-para-acesso-via-ssh/
* https://www.digitalocean.com/community/tutorials/configuracao-inicial-de-servidor-com-ubuntu-16-04-pt
* https://git-scm.com/book/pt-br/v1/Customizando-o-Git-Hooks-do-Git
* https://www.digitalocean.com/community/tutorials/como-instalar-o-node-js-no-ubuntu-16-04-pt