# Deploy de sistema Django com Docker Compose

Este tutorial apresenta como fazer o deploy de uma aplicação Django em uma VM (ex. Google Cloud) utilizando Docker Compose com Gunicorn, PostgreSQL e Nginx.

!!! warning "Atenção!"
    Antes de seguir este tutorial, certifique-se de que você estudou o conteúdo da aula sobre conteinerização e Docker. Este tutorial assume que você já tem uma VM criada ([Tutorial 02](tutorial02.md)) com o Docker instalado e funcionando.

## Visão Geral

O sistema é dividido em três containers, gerenciados pelo Docker Compose:

- **nginx**: recebe as requisições do navegador, entrega os arquivos estáticos e de *media* e repassa o restante para o container `web`;
- **web**: executa o Django com o Gunicorn;
- **db**: banco de dados PostgreSQL.

Os exemplos usam os seguintes nomes, que você deve substituir pelos seus:

| Nome | Significado |
|---|---|
| `<projeto>` | nome da pasta do seu projeto Django |
| `<banco-do-app>` | nome do banco de dados |
| `<user-do-app>` | usuário do banco de dados |
| `<senha-do-user-do-app>` | senha do usuário do banco de dados |
| `<ip-externo>` | IP externo da VM, exibido no console do Google Cloud |

O projeto deve ter a pasta de configuração chamada `config` (a que contém `settings.py` e `wsgi.py`). Se a sua tiver outro nome, ajuste os arquivos onde aparece `config`.

## Parte 1: Preparação na máquina local

Todas as alterações no código são feitas na sua máquina e enviadas para o servidor pelo Git. Ao final desta parte, o projeto terá a seguinte estrutura:

```
<projeto>/
├── config/              # pasta de configuração do Django
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── app/                 # suas apps Django
├── manage.py
├── requirements.txt
├── requirements-prod.txt
├── Dockerfile
├── docker-compose.yml
├── entrypoint.sh
├── .dockerignore
├── .env.exemplo
└── nginx/
    └── nginx.conf
```

### Dependências

O `python-dotenv` lê as configurações de um arquivo `.env`. Com o *venv* do projeto ativado, instale-o e atualize o arquivo de dependências:

```bash
pip install python-dotenv
pip freeze > requirements.txt
```

Depois, crie o arquivo `requirements-prod.txt`, com as dependências usadas apenas no container:

```txt
-r requirements.txt
gunicorn>=26.0
psycopg[binary]>=3.3
```

A linha `-r requirements.txt` importa todas as dependências do projeto, então instalar o `requirements-prod.txt` instala tudo. O `psycopg` é o driver que permite ao Django se comunicar com o PostgreSQL.

!!! warning "Atenção!"
    Se o `requirements.txt` foi gerado no Windows, remova pacotes exclusivos dele (ex. `pywin32`), senão a instalação falha no container.

### Modificando o settings.py

Altere o início do arquivo `config/settings.py`:

```python
import os
from pathlib import Path
from dotenv import load_dotenv, find_dotenv

load_dotenv(find_dotenv())

BASE_DIR = Path(__file__).resolve().parent.parent

SECRET_KEY = os.getenv("SECRET_KEY", "django-insecure-chave-local")

DEBUG = os.getenv("DEBUG", "True") == "True"

ALLOWED_HOSTS = os.getenv("ALLOWED_HOSTS", "localhost,127.0.0.1").split(",")
```

Substitua a configuração `DATABASES`:

```python
if os.getenv("DB_ENGINE") == "postgresql":  # PostgreSQL
    DATABASES = {
        "default": {
            "ENGINE": "django.db.backends.postgresql",
            "NAME": os.getenv("DB_NAME"),
            "USER": os.getenv("DB_USER"),
            "PASSWORD": os.getenv("DB_PASSWORD"),
            "HOST": "db",
            "PORT": "5432",
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

O `HOST` é `db`, o nome do serviço do banco no `docker-compose.yml`.

E configure os arquivos estáticos e de *media* (no final do arquivo):

```python
STATIC_URL = "static/"
STATIC_ROOT = os.getenv("STATIC_ROOT", BASE_DIR / "staticfiles")

MEDIA_URL = "media/"
MEDIA_ROOT = os.getenv("MEDIA_ROOT", BASE_DIR / "media")
```

Mantenha o restante do `settings.py` (`INSTALLED_APPS`, `MIDDLEWARE`, `TEMPLATES` etc.) como está.

!!! info "Como funciona?"
    As configurações de produção são lidas de variáveis de ambiente, definidas no arquivo `.env` do servidor, que nunca vai para o repositório. O Docker Compose repassa as variáveis do `.env` para os containers.

### Arquivo .env.exemplo

Como o `.env` não vai para o repositório, crie na raiz do projeto um arquivo `.env.exemplo` que serve de modelo. Ele lista todas as variáveis usadas pelo projeto, com valores fictícios:

```env
DEBUG=False
SECRET_KEY=<chave-gerada>
ALLOWED_HOSTS=<ip-externo>
DB_ENGINE=postgresql
DB_NAME=<banco-do-app>
DB_USER=<user-do-app>
DB_PASSWORD=<senha-do-user-do-app>
```

O `.env.exemplo` vai para o repositório, então nunca coloque senhas ou chaves reais nele. Sempre que o projeto passar a usar uma nova variável, adicione-a também ao `.env.exemplo`.

### Dockerfile

Crie o arquivo `Dockerfile` na raiz do projeto:

```dockerfile
FROM python:3.14-slim

# Não gera arquivos .pyc e exibe os logs do Python imediatamente
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

# Diretório de trabalho
WORKDIR /app

# Copia e instala dependências Python
COPY requirements.txt requirements-prod.txt ./
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir -r requirements-prod.txt

# Script de inicialização
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

# Copia código da aplicação
COPY . .

# Porta do Gunicorn
EXPOSE 8000

ENTRYPOINT ["/entrypoint.sh"]
```

!!! info "Versão do Python"
    O Django 6 requer Python 3.12 ou superior. Se o seu projeto usa uma versão mais antiga do Django, verifique a versão do Python compatível e ajuste a imagem base (ex. `python:3.12-slim`).

### Script de inicialização (entrypoint.sh)

Crie o arquivo `entrypoint.sh` na raiz do projeto:

```bash
#!/bin/bash
set -e  # interrompe o script se algum comando falhar

# O docker-compose.yml só inicia este container quando o PostgreSQL
# estiver pronto (healthcheck), então as migrations podem rodar direto.

# Executa migrations
python manage.py migrate --noinput

# Coleta arquivos estáticos
python manage.py collectstatic --noinput

# Inicia Gunicorn
exec gunicorn config.wsgi:application \
    --bind 0.0.0.0:8000 \
    --workers 3
```

!!! warning "Atenção!"
    Se você estiver no Windows, certifique-se de que o arquivo `entrypoint.sh` está com quebras de linha no formato Unix (LF) e não Windows (CRLF). Você pode usar o VS Code para alterar isso no canto inferior direito.

### docker-compose.yml

Crie o arquivo `docker-compose.yml` na raiz do projeto:

```yaml
services:
  db:
    image: postgres:18
    container_name: django_db
    volumes:
      - postgres_data:/var/lib/postgresql
    environment:
      - POSTGRES_DB=${DB_NAME}
      - POSTGRES_USER=${DB_USER}
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER} -d $${POSTGRES_DB}"]
      interval: 5s
      timeout: 5s
      retries: 10
    networks:
      - django_network
    restart: unless-stopped

  web:
    build: .
    container_name: django_web
    env_file: .env
    volumes:
      - static_volume:/app/staticfiles
      - media_volume:/app/media
    depends_on:
      db:
        condition: service_healthy
    networks:
      - django_network
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    container_name: django_nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - static_volume:/app/staticfiles:ro
      - media_volume:/app/media:ro
    depends_on:
      - web
    networks:
      - django_network
    restart: unless-stopped

volumes:
  postgres_data:
  static_volume:
  media_volume:

networks:
  django_network:
    driver: bridge
```

O serviço `web` recebe todas as variáveis do `.env` através do `env_file`. O serviço `db` usa os mesmos valores para criar o banco e o usuário na primeira execução, através das variáveis `POSTGRES_*` exigidas pela imagem do PostgreSQL.

O `healthcheck` do serviço `db` usa o comando `pg_isready` para verificar se o banco já aceita conexões. Com `condition: service_healthy`, o container `web` só inicia depois disso, evitando erros de conexão ao rodar as migrations.

!!! warning "PostgreSQL 18"
    A imagem do PostgreSQL 18 guarda os dados em `/var/lib/postgresql/18/docker`, por isso o volume é montado em `/var/lib/postgresql`. Assim os dados do banco ficam no volume e não se perdem quando o container é recriado.

### Configuração do Nginx (nginx/nginx.conf)

Crie o diretório `nginx` e o arquivo `nginx.conf` dentro dele:

```nginx
events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    upstream django {
        server web:8000;
    }

    server {
        listen 80;
        server_name _;
        client_max_body_size 10M;

        location /static/ {
            alias /app/staticfiles/;
        }

        location /media/ {
            alias /app/media/;
        }

        location / {
            proxy_pass http://django;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

O `client_max_body_size` aumenta o limite de upload do Nginx, que é de 1 MB por padrão.

### .dockerignore

Crie o arquivo `.dockerignore` na raiz do projeto para excluir arquivos desnecessários da imagem:

```
*.pyc
__pycache__
*.pyo
*.pyd
.Python
env/
venv/
.venv/
.git
.gitignore
.dockerignore
*.md
.env
db.sqlite3
*.log
.coverage
htmlcov/
.pytest_cache/
staticfiles/
media/
```

O `.env` fica fora da imagem: as variáveis chegam ao container pelo Docker Compose.

### Migrations

As *migrations* são criadas na máquina local e enviadas pelo Git. No servidor, elas são apenas aplicadas (pelo `entrypoint.sh`). Antes de enviar o projeto, crie as que estiverem faltando:

```bash
python manage.py makemigrations
```

### Git

Verifique se o `.gitignore` contém as seguintes linhas:

```
.env
venv/
db.sqlite3
staticfiles/
media/
```

A linha `.env` ignora apenas o arquivo `.env`. O `.env.exemplo` continua indo para o repositório.

Faça o *commit* e envie as alterações para o repositório (`git push`).

## Parte 2: Configuração do servidor

Acesse o servidor via SSH para os próximos passos.

### Copiando o projeto

Clone o repositório na sua *home*:

```bash
cd ~
git clone <url-do-repositorio> <projeto>
cd <projeto>
```

### Arquivo .env

Gere uma `SECRET_KEY` para o servidor:

```bash
python3 -c "import secrets; print(secrets.token_urlsafe(50))"
```

Crie o `.env` a partir do modelo `.env.exemplo` e abra-o para edição:

```bash
cp .env.exemplo .env
nano .env
```

Substitua os valores entre `< >` pelos dados reais: a chave gerada, o IP externo da VM e os dados do banco. O banco e o usuário são criados automaticamente pelo container `db` com esses valores.

Proteja o arquivo para que apenas o seu usuário possa lê-lo:

```bash
chmod 600 .env
```

!!! warning "IP externo"
    No Google Cloud, o IP externo muda quando a VM é desligada e ligada novamente. Quando isso acontecer, atualize o `ALLOWED_HOSTS` no `.env` e recrie os containers com `docker compose up -d`.

### Executando o projeto

Construa as imagens e inicie os containers:

```bash
docker compose up -d --build
```

Verifique se os containers estão rodando. O `db` deve aparecer como `healthy`:

```bash
docker compose ps
```

Acesse `http://<ip-externo>` no navegador. O sistema deve estar funcionando, inclusive o `/admin` com seus estilos.

### Superusuário e verificação

Crie o superusuário do Django:

```bash
docker compose exec web python manage.py createsuperuser
```

Verifique a configuração de produção:

```bash
docker compose exec web python manage.py check --deploy
```

O comando lista problemas de segurança e configuração:

- Avisos sobre HTTPS (`SECURE_SSL_REDIRECT`, `SESSION_COOKIE_SECURE` etc.) são esperados, pois este tutorial não configura HTTPS;
- O erro `mail.E001` indica que o envio de email não foi configurado (veja a seção Email do [Tutorial 03](tutorial09.md#email-opcional));
- Avisos sobre `SECRET_KEY` ou `DEBUG` indicam que o `.env` não foi lido corretamente.

## Comandos Úteis

### Gerenciamento dos Containers

```bash
# Parar os containers
docker compose down

# Parar e remover os volumes (apaga dados do banco!)
docker compose down -v

# Reiniciar apenas um serviço
docker compose restart web

# Ver status dos serviços
docker compose ps
```

### Executando Comandos no Container

```bash
# Acessar o shell do container web
docker compose exec web bash

# Acessar o shell do Django
docker compose exec web python manage.py shell

# Acessar o banco de dados
docker compose exec db psql -U <user-do-app> <banco-do-app>
```

### Logs

```bash
# Ver logs de todos os serviços
docker compose logs

# Ver logs de um serviço específico
docker compose logs web
docker compose logs db
docker compose logs nginx

# Seguir os logs em tempo real
docker compose logs -f web
```

## Backup do Banco de Dados

### Criar Backup

```bash
docker compose exec -T db pg_dump -U <user-do-app> <banco-do-app> > backup.sql
```

### Restaurar Backup

```bash
docker compose exec -T db psql -U <user-do-app> <banco-do-app> < backup.sql
```

## Atualizando o Projeto

Faça as alterações na máquina local, crie as *migrations* se necessário e envie para o repositório. No servidor:

```bash
cd ~/<projeto>
git pull
docker compose up -d --build
```

Não é preciso rodar `migrate` e `collectstatic` manualmente: o `entrypoint.sh` faz isso toda vez que o container `web` inicia.

Ao alterar apenas o `.env`, execute `docker compose up -d`. O `docker compose restart` não relê o `.env`.

## Solução de Problemas

### Onde ver os erros

```bash
# Status dos containers
docker compose ps

# Erros do Django/Gunicorn
docker compose logs web

# Erros do Nginx
docker compose logs nginx
```

### Erros comuns

| Erro | Causa provável |
|---|---|
| 502 Bad Gateway | O container `web` não está rodando. Verifique o `docker compose logs web` |
| 400 Bad Request | O IP ou domínio acessado não está no `ALLOWED_HOSTS` do `.env` |
| 413 Request Entity Too Large | Upload maior que o `client_max_body_size` do Nginx |
| Página sem estilos (CSS) | O `collectstatic` falhou (veja `docker compose logs web`) ou os volumes `static_volume` não correspondem ao `STATIC_ROOT` |
| Container `web` reiniciando | Erro no `entrypoint.sh` ou nas migrations. Veja `docker compose logs web` |
| `exec /entrypoint.sh: no such file or directory` | O `entrypoint.sh` está com quebras de linha do Windows (CRLF) |
| Erro de acesso ao banco | O `db` não está `healthy` ou os dados do `.env` foram alterados depois da criação do volume |

!!! info "Dados do banco"
    O banco e o usuário são criados apenas na primeira execução, quando o volume `postgres_data` está vazio. Alterar `DB_NAME`, `DB_USER` ou `DB_PASSWORD` depois disso não altera o banco existente.

## Considerações de Produção

Para uso em produção, considere:

- Configurar HTTPS com certificados SSL (Let's Encrypt)
- Usar senhas fortes e únicas
- Implementar backup automatizado do banco de dados
- Configurar monitoramento e alertas
- Usar um registry privado para as imagens Docker
- Configurar health checks também nos containers `web` e `nginx`
- Ao usar HTTPS atrás do Nginx, adicionar `SECURE_PROXY_SSL_HEADER = ("HTTP_X_FORWARDED_PROTO", "https")` ao `settings.py`
