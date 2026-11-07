# Deploy de sistema Django

Este tutorial apresenta como fazer o deploy de uma aplicação Django em uma VM Debian (ex. Google Cloud) utilizando Gunicorn, Nginx e MariaDB.

!!! warning "Atenção!"
    Antes de seguir este tutorial, certifique-se de que você estudou o conteúdo da aula sobre deploy de sistemas Django. Este tutorial assume que você já tem uma VM criada ([Tutorial 02](tutorial02.md)) com o Nginx e o MariaDB instalados e funcionando.

## Visão Geral

No servidor, cada parte do sistema tem uma função:

- **Nginx**: recebe as requisições do navegador, entrega os arquivos estáticos e de *media* e repassa o restante para o Gunicorn;
- **Gunicorn**: executa o Django e se comunica com o Nginx através de um *socket*;
- **systemd**: mantém o Gunicorn rodando como um serviço (*daemon*);
- **MariaDB**: banco de dados da aplicação.

Os exemplos usam os seguintes nomes, que você deve substituir pelos seus:

| Nome | Significado |
|---|---|
| `<projeto>` | nome da pasta do seu projeto Django |
| `<user>` | seu usuário no servidor (descubra com o comando `whoami`) |
| `<banco-do-app>` | nome do banco de dados |
| `<user-do-app>` | usuário do banco de dados |
| `<senha-do-user-do-app>` | senha do usuário do banco de dados |
| `<ip-externo>` | IP externo da VM, exibido no console do Google Cloud |

O projeto deve ter a pasta de configuração chamada `config` (a que contém `settings.py` e `wsgi.py`). Se a sua tiver outro nome, ajuste os comandos onde aparece `config`.

## Parte 1: Preparação na máquina local

Todas as alterações no código são feitas na sua máquina e enviadas para o servidor pelo Git.

### Instalando o python-dotenv

O `python-dotenv` lê as configurações de produção de um arquivo `.env`. Com o *venv* do projeto ativado, instale-o:

```bash
pip install python-dotenv
```

### Modificando o settings.py

Altere o início do arquivo `config/settings.py`:

```python
import os
from pathlib import Path
from dotenv import load_dotenv

BASE_DIR = Path(__file__).resolve().parent.parent
load_dotenv(BASE_DIR / ".env")

SECRET_KEY = os.environ.get("SECRET_KEY", "django-insecure-chave-local")

DEBUG = os.environ.get("DEBUG", "True") == "True"

ALLOWED_HOSTS = os.environ.get("ALLOWED_HOSTS", "localhost,127.0.0.1").split(",")
```

Substitua a configuração `DATABASES`:

```python
if os.environ.get("DB_ENGINE") == "mysql":  # MariaDB
    DATABASES = {
        "default": {
            "ENGINE": "django.db.backends.mysql",
            "OPTIONS": {"sql_mode": "traditional"},
            "NAME": os.environ.get("DB_NAME"),
            "USER": os.environ.get("DB_USER"),
            "PASSWORD": os.environ.get("DB_PASSWORD"),
            "HOST": "localhost",
            "PORT": "3306",
        }
    }
else:  # SQLite
    DATABASES = {
        "default": {
            "ENGINE": "django.db.backends.sqlite3",
            "NAME": BASE_DIR / "db.sqlite3",
        }
    }
```

E configure os arquivos estáticos e de *media* (no final do arquivo):

```python
STATIC_URL = "static/"
STATIC_ROOT = os.environ.get("STATIC_ROOT", BASE_DIR / "staticfiles")

MEDIA_URL = "media/"
MEDIA_ROOT = os.environ.get("MEDIA_ROOT", BASE_DIR / "media")
```

Mantenha o restante do `settings.py` (`INSTALLED_APPS`, `MIDDLEWARE`, `TEMPLATES` etc.) como está.

!!! info "Como funciona?"
    As configurações de produção são lidas de variáveis de ambiente, definidas no arquivo `.env` do servidor, que nunca vai para o repositório.

### Migrations

As *migrations* são criadas na máquina local e enviadas pelo Git. No servidor, elas são apenas aplicadas. Antes de enviar o projeto, crie as que estiverem faltando:

```bash
python manage.py makemigrations
```

### Dependências e Git

Atualize o arquivo de dependências:

```bash
pip freeze > requirements.txt
```

!!! warning "Atenção!"
    Se o `requirements.txt` foi gerado no Windows, remova pacotes exclusivos dele (ex. `pywin32`), senão a instalação falha no servidor.

Verifique se o `.gitignore` contém as seguintes linhas:

```
.env
venv/
db.sqlite3
staticfiles/
media/
```

Faça o *commit* e envie as alterações para o repositório (`git push`).

## Parte 2: Configuração do servidor

Acesse o servidor via SSH para os próximos passos.

### Dependências do sistema

```bash
sudo apt update
sudo apt install python3-venv python3-dev build-essential libmariadb-dev pkg-config git
```

Os pacotes `python3-dev`, `build-essential`, `libmariadb-dev` e `pkg-config` são necessários para compilar o `mysqlclient`, o driver que conecta o Django ao MariaDB.

!!! info "Versão do Python"
    Verifique a versão do Python do servidor com `python3 --version`. O Django 6 requer Python 3.12 ou superior (Debian 13). No Debian 12 (Python 3.11), use o Django 5.2.

### Copiando o projeto

Clone o repositório na sua *home*:

```bash
cd ~
git clone <url-do-repositorio> <projeto>
cd <projeto>
```

### Ambiente virtual

Crie o *venv* dentro da pasta do projeto e instale as dependências, incluindo as que só são usadas no servidor:

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
pip install mysqlclient gunicorn
```

!!! warning "Atenção!"
    Nunca copie o *venv* da sua máquina para o servidor. Crie um novo em cada máquina.

### Banco de dados

Acesse o MariaDB como administrador:

```bash
sudo mariadb
```

Crie o banco e o usuário da aplicação:

```sql
CREATE DATABASE <banco-do-app> CHARACTER SET utf8mb4;
CREATE USER '<user-do-app>'@'localhost' IDENTIFIED BY '<senha-do-user-do-app>';
GRANT ALL PRIVILEGES ON <banco-do-app>.* TO '<user-do-app>'@'localhost';
quit
```

### Arquivo .env

Gere uma `SECRET_KEY` para o servidor:

```bash
python -c "import secrets; print(secrets.token_urlsafe(50))"
```

Crie o arquivo `.env` na pasta do projeto (`nano .env`):

```env
DEBUG=False
SECRET_KEY=<chave-gerada>
ALLOWED_HOSTS=<ip-externo>
DB_ENGINE=mysql
DB_NAME=<banco-do-app>
DB_USER=<user-do-app>
DB_PASSWORD=<senha-do-user-do-app>
STATIC_ROOT=/var/www/html/<projeto>/static/
MEDIA_ROOT=/var/www/html/<projeto>/media/
```

Proteja o arquivo para que apenas o seu usuário possa lê-lo:

```bash
chmod 600 .env
```

!!! warning "IP externo"
    No Google Cloud, o IP externo muda quando a VM é desligada e ligada novamente. Quando isso acontecer, atualize o `ALLOWED_HOSTS` no `.env` e reinicie o Gunicorn (veja [Atualizando o Projeto](#atualizando-o-projeto)).

### Arquivos estáticos e media

Crie os diretórios e mude o proprietário para o seu usuário:

```bash
sudo mkdir -p /var/www/html/<projeto>/static /var/www/html/<projeto>/media
sudo chown -R <user>:<user> /var/www/html/<projeto>
```

### Migrations, estáticos e superusuário

Com o *venv* ativado:

```bash
python manage.py migrate
python manage.py collectstatic
python manage.py createsuperuser
```

Verifique a configuração de produção:

```bash
python manage.py check --deploy
```

O comando lista problemas de segurança e configuração:

- Avisos sobre HTTPS (`SECURE_SSL_REDIRECT`, `SESSION_COOKIE_SECURE` etc.) são esperados, pois este tutorial não configura HTTPS;
- O erro `mail.E001` indica que o envio de email não foi configurado (veja [Email](#email-opcional));
- Avisos sobre `SECRET_KEY` ou `DEBUG` indicam que o `.env` não foi lido corretamente.

## Parte 3: Gunicorn

O Gunicorn é configurado como um serviço do systemd, com um *socket* por onde o Nginx se comunica com ele.

### Socket

Crie o arquivo do *socket*:

```bash
sudo nano /etc/systemd/system/<projeto>_gunicorn.socket
```

Com o seguinte conteúdo:

```ini
[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/<projeto>_gunicorn.sock

[Install]
WantedBy=sockets.target
```

### Serviço

Crie o arquivo do serviço:

```bash
sudo nano /etc/systemd/system/<projeto>_gunicorn.service
```

Com o seguinte conteúdo:

```ini
[Unit]
Description=gunicorn service
After=network.target
Requires=<projeto>_gunicorn.socket

[Service]
Type=notify
NotifyAccess=main
User=<user>
Group=www-data
WorkingDirectory=/home/<user>/<projeto>/
ExecStart=/home/<user>/<projeto>/venv/bin/gunicorn \
          --access-logfile - --workers 3 \
          --bind unix:/run/<projeto>_gunicorn.sock \
          config.wsgi:application
ExecReload=/bin/kill -s HUP $MAINPID

[Install]
WantedBy=multi-user.target
```

### Habilitando o serviço

```bash
sudo systemctl enable --now <projeto>_gunicorn.socket
sudo systemctl enable --now <projeto>_gunicorn.service
sudo systemctl status <projeto>_gunicorn.service
```

O status deve mostrar `active (running)`. Teste se o Django responde pelo *socket*:

```bash
curl --unix-socket /run/<projeto>_gunicorn.sock http://localhost/
```

O comando deve exibir o HTML da página inicial do seu sistema.

## Parte 4: Nginx

Crie o arquivo de configuração do site:

```bash
sudo nano /etc/nginx/sites-available/<projeto>
```

Com o seguinte conteúdo:

```nginx
server {
    listen 80;
    server_name _;
    client_max_body_size 10M;

    location = /favicon.ico { access_log off; log_not_found off; }

    location /static/ {
        alias /var/www/html/<projeto>/static/;
    }

    location /media/ {
        alias /var/www/html/<projeto>/media/;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/run/<projeto>_gunicorn.sock;
    }
}
```

O `client_max_body_size` aumenta o limite de upload do Nginx, que é de 1 MB por padrão.

Habilite o site e desabilite o site padrão do Nginx:

```bash
sudo ln -s /etc/nginx/sites-available/<projeto> /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default
```

!!! info "Site padrão"
    O comando `rm` remove apenas o atalho em `sites-enabled`. O arquivo original continua em `sites-available/default`.

Verifique a configuração e recarregue o Nginx:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

Acesse `http://<ip-externo>` no navegador. O sistema deve estar funcionando, inclusive o `/admin` com seus estilos.

## Atualizando o Projeto

Faça as alterações na máquina local, crie as *migrations* se necessário e envie para o repositório. No servidor:

```bash
cd ~/<projeto>
git pull
source venv/bin/activate
pip install -r requirements.txt
python manage.py migrate
python manage.py collectstatic --noinput
sudo systemctl restart <projeto>_gunicorn.service
```

Ao alterar apenas o `.env`, basta reiniciar o Gunicorn (último comando).

!!! info "systemd"
    Se você alterar os arquivos `.socket` ou `.service`, execute `sudo systemctl daemon-reload` antes de reiniciar o serviço.

## Email (opcional)

Servidores de email novos costumam cair em filtros de *spam*, então o ideal é enviar emails através do SMTP de um provedor conhecido. Exemplo com o Gmail, que exige a verificação em duas etapas e uma [senha de app](https://myaccount.google.com/apppasswords):

```python
if os.environ.get("EMAIL_USER"):  # envia pelo Gmail
    MAILERS = {
        "default": {
            "BACKEND": "django.core.mail.backends.smtp.EmailBackend",
            "OPTIONS": {
                "host": "smtp.gmail.com",
                "port": 587,
                "use_tls": True,
                "username": os.environ.get("EMAIL_USER"),
                "password": os.environ.get("EMAIL_PASSWORD"),
            },
        },
    }
else:  # exibe os emails no terminal
    MAILERS = {
        "default": {
            "BACKEND": "django.core.mail.backends.console.EmailBackend",
        },
    }
```

E no `.env` do servidor:

```env
EMAIL_USER=seuemail@gmail.com
EMAIL_PASSWORD=<senha-de-app>
```

!!! warning "Versões do Django"
    O `MAILERS` foi introduzido no Django 6.1 e substitui as configurações `EMAIL_BACKEND`, `EMAIL_HOST`, `EMAIL_HOST_USER` etc., que não podem ser usadas junto com ele. Remova-as do `settings.py`. Em versões anteriores do Django, use as configurações `EMAIL_*` ([documentação](https://docs.djangoproject.com/en/5.2/topics/email/#smtp-backend)).

## Solução de Problemas

### Onde ver os erros

```bash
# Erros do Django/Gunicorn
sudo journalctl -u <projeto>_gunicorn.service -f

# Erros do Nginx
sudo tail /var/log/nginx/error.log
```

### Erros comuns

| Erro | Causa provável |
|---|---|
| Página padrão do Nginx | O site `default` ainda está habilitado em `sites-enabled` |
| 502 Bad Gateway | O Gunicorn não está rodando. Verifique o `systemctl status` e o `journalctl` |
| 400 Bad Request | O IP ou domínio acessado não está no `ALLOWED_HOSTS` do `.env` |
| 413 Request Entity Too Large | Upload maior que o `client_max_body_size` do Nginx |
| Página sem estilos (CSS) | O `collectstatic` não foi executado ou o `alias` do Nginx não corresponde ao `STATIC_ROOT` |
| Erro ao instalar o `mysqlclient` | Faltam os pacotes do sistema (`python3-dev`, `build-essential`, `libmariadb-dev`, `pkg-config`) |
| Erro de acesso ao banco | Dados do `.env` não correspondem ao usuário e banco criados no MariaDB |
