# Servidor de Aplicação Linux com Nginx & Gunicorn
## Descrição do Projeto

Este projeto demonstra a configuração de um ambiente de produção completo para uma aplicação web backend, utilizando um stack de tecnologias comum no mercado. O objetivo principal é servir uma aplicação Flask com o servidor de aplicação Gunicorn, e gerenciar o acesso externo e o balanceamento de carga com o Nginx, tudo rodando em uma máquina virtual Linux CentOS.

### O projeto foi desenvolvido para consolidar conhecimentos em:

Administração de Sistemas Linux: Gerenciamento de pacotes, arquivos, e permissões via linha de comando.

Servidores de Aplicação: Compreensão do papel do Gunicorn na execução de aplicações Python em ambientes de produção.

Proxy Reverso e Servidor Web: Utilização do Nginx para otimizar o acesso à aplicação, servindo como um proxy reverso robusto e seguro.

Gerenciamento de Serviços: Configuração e controle de processos de background usando systemd.

### Tecnologias Utilizadas

Sistema Operacional: CentOS

Linguagem de Programação: Python

Framework Web: Flask

Servidor de Aplicação: Gunicorn

Servidor Web / Proxy Reverso: Nginx

Gerenciamento de Serviços: systemd

## Instalação e Configuração
### Siga os passos abaixo para replicar o ambiente do projeto em uma máquina virtual CentOS.


### 1. Preparando o Ambiente do Sistema

#### Atualize o sistema
`sudo yum update
`
#### Adicione o repositório EPEL
`sudo yum install epel-release
`
#### Instale o Nginx e dependências do Python
`sudo yum install python-pip python-devel gcc nginx
` 
### 2. Configurando a Aplicação Flask

#### Instale o virtualenv e crie a pasta do projeto
`sudo pip install virtualenv`
#### Crie e acesse sua pasta de projetos
```
cd /home/user/Documents/python
mkdir myproject
cd myproject
```
#### Crie e ative o ambiente virtual
```
virtualenv myprojectenv
source myprojectenv/bin/activate
```
#### Instale as dependências da aplicação
`pip install gunicorn flask`

#### Crie o arquivo da sua aplicação (por exemplo, app.py)
`vi app.py`

#### Cole o seguinte código no arquivo app.py:

```
from flask import Flask
application = Flask(__name__)

@application.route("/")
def hello():
    return "<h1 style='color:blue'>Hello World from Nome do Aluno!</h1>"

if __name__ == "__main__":
    application.run(host='0.0.0.0')
```
### 3. Configurando o Gunicorn

#### Crie o arquivo de inicialização do Gunicorn (wsgi.py)
`vi wsgi.py`
#### Cole o seguinte conteúdo no arquivo wsgi.py:

```
from app import application

if __name__ == "__main__":
    application.run()

```

### 4. Configurando como Serviço do Sistema

#### Desative o ambiente virtual
`deactivate`


#### Crie o arquivo de serviço do Gunicorn
`sudo nano /etc/systemd/system/myproject.service`

#### Cole o seguinte conteúdo no arquivo myproject.service, substituindo myproject e myprojectenv pelos nomes do seu projeto e ambiente virtual, respectivamente.

```
[Unit]
Description=Gunicorn instance to serve myproject
After=network.target

[Service]
User=user
Group=nginx
WorkingDirectory=/home/user/Documents/python/myproject
Environment="PATH=/home/user/Documents/python/myproject/myprojectenv/bin"
ExecStart=/home/user/Documents/python/myproject/myprojectenv/bin/gunicorn --workers 3 --bind unix:myproject.sock -m 007 wsgi:app

[Install]
WantedBy=multi-user.target
```bash
# Habilite e inicie o serviço
sudo systemctl start myproject
sudo systemctl enable myproject
```

### 5. Configurando o Nginx

#### Abra o arquivo de configuração do Nginx
`sudo nano /etc/nginx/nginx.conf`


#### Substitua o bloco server {} pelo seguinte, ajustando os caminhos para o seu projeto:

```
server {
    listen       80;
    listen       [::]:80 default_server;
    server_name  myproject www.myproject;

    # Logs
    access_log /var/log/nginx/myproject.access.log;
    error_log /var/log/nginx/myproject.error.log;

    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;

    location / {
        proxy_pass http://unix:/home/user/Documents/python/myproject/myproject.sock;
    }

    error_page 404 /404.html;
    location = /40x.html {
    }

    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
    }
}
```bash
# Altere as permissões para o Nginx acessar a pasta do projeto
sudo usermod -a -G user nginx
sudo chmod 710 /home/user

# Verifique a sintaxe da configuração e reinicie o Nginx
sudo nginx -t
sudo systemctl restart nginx
sudo systemctl enable nginx

```
### 6. Teste e Acesso

#### Obtenha o IP da sua máquina virtual
`ifconfig`

#### Edite o arquivo /etc/hosts para mapear o IP ao nome do seu projeto:

`sudo nano /etc/hosts
`
#### Adicione a seguinte linha (substituindo o IP e o nome do seu projeto):
`10.0.2.15   myproject
`
#### Agora, reinicie os serviços e acesse a sua aplicação no navegador.
`sudo systemctl restart myproject`
`sudo systemctl restart nginx`

Abra o Firefox na sua máquina virtual e acesse: http://myproject



## Conclusão

#### Este projeto demonstra a capacidade de configurar e gerenciar um ambiente de produção real, indo além do simples desenvolvimento de código. A automação com systemd e a otimização de tráfego com Nginx são práticas essenciais para a área de backend e DevOps. Este é um excelente ponto de partida para projetos mais complexos que exijam escalabilidade e robustez.
