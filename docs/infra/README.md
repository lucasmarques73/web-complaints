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

### Configuração de VHost

Configurei dois vhosts para testes, usando um como um subdominio.  
As configurações são básicamente as mesmas diferenciando dois paramêtros.

Alterando para usuário `root`

```
sudo su
```

Criando o arquivo.

```
touch /etc/nginx/sites-available/home
```

Editando o arquivo.

```
nano /etc/nginx/sites-available/home
```
```
server{
	server_name www.lucasmarques73.com.br lucasmarques73.com.br;
	root /var/www/html;
	
	index index.html;

	charset utf-8;

	location / {
		try_files $uri $uri/ =404;
	}

}
```

* Lembrando que temos que ter dentro de `var/www/html` um arquivo com o nome de `idex.html` que será nossa pagina inicial.

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
ln -s /etc/nginx/sites-available/home /etc/nginx/sites-enabled/home
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