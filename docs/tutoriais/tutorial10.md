# Deploy de sistema Django com Docker Compose

Este tutorial apresenta como fazer o deploy de uma aplicação Django utilizando Docker Compose com PostgreSQL e Nginx.

!!! warning "Atenção!"
    Antes de seguir este tutorial, certifique-se de que você estudou o conteúdo da aula sobre conteinerização e Docker. Este tutorial assume que você já tem o Docker instalado e funcionando no seu sistema.

## Estrutura do Projeto

Crie a seguinte estrutura de arquivos na raiz do seu projeto Django:

```
projeto/
├── config/              # pasta de configuração do Django (pode ter outro nome)
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── app/                 # suas apps Django
├── manage.py
├── requirements.txt
├── Dockerfile
├── docker-compose.yml
├── entrypoint.sh
├── .dockerignore
├── .env
└── nginx/
    └── nginx.conf
```

## Dockerfile

Crie o arquivo `Dockerfile` na raiz do projeto:

```dockerfile
FROM python:3.14-slim

# Não gera arquivos .pyc e exibe os logs do Python imediatamente
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

# Diretório de trabalho
WORKDIR /app

# Copia e instala dependências Python
COPY requirements.txt .
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir -r requirements.txt

# Script de inicialização
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

# Copia código da aplicação
COPY . .

# Porta do Gunicorn
EXPOSE 8000

ENTRYPOINT ["/entrypoint.sh"]
```

## Script de Inicialização (entrypoint.sh)

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

## requirements.txt

Adicione as dependências necessárias ao seu `requirements.txt`:

```txt
Django>=6.1,<6.2
psycopg[binary]>=3.3
gunicorn>=26.0
```

Adicione outras dependências do seu projeto conforme necessário.

!!! info "Observações"
    - O `psycopg` é o driver que permite ao Django se comunicar com o PostgreSQL.
    - O Django 6.1 exige Python 3.12 ou superior. Se o seu projeto usa a versão LTS (5.2), troque a primeira linha por `Django>=5.2,<5.3`.
    - As variáveis do arquivo `.env` são lidas pelo Docker Compose e repassadas para o container, então o Django as acessa diretamente com `os.environ`.

## docker-compose.yml

Crie o arquivo `docker-compose.yml` na raiz do projeto:

```yaml
services:
  db:
    image: postgres:18
    container_name: django_db
    volumes:
      - postgres_data:/var/lib/postgresql
    environment:
      - POSTGRES_DB=${POSTGRES_DB}
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
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
    volumes:
      - static_volume:/app/staticfiles
      - media_volume:/app/media
    environment:
      - DEBUG=${DEBUG}
      - SECRET_KEY=${SECRET_KEY}
      - ALLOWED_HOSTS=${ALLOWED_HOSTS}
      - CSRF_TRUSTED_ORIGINS=${CSRF_TRUSTED_ORIGINS}
      - POSTGRES_DB=${POSTGRES_DB}
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - DB_HOST=db
      - DB_PORT=5432
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

O `healthcheck` do serviço `db` usa o comando `pg_isready` para verificar se o banco já aceita conexões. Com `condition: service_healthy`, o container `web` só inicia depois disso, evitando erros de conexão ao rodar as migrations.

!!! warning "PostgreSQL 18"
    A imagem do PostgreSQL 18 guarda os dados em `/var/lib/postgresql/18/docker`, por isso o volume é montado em `/var/lib/postgresql`. Assim os dados do banco ficam no volume e não se perdem quando o container é recriado.

## Arquivo de Variáveis de Ambiente (.env)

Crie o arquivo `.env` na raiz do projeto:

```env
DEBUG=False
SECRET_KEY=sua-chave-secreta-muito-longa-e-segura-aqui
ALLOWED_HOSTS=localhost,127.0.0.1
CSRF_TRUSTED_ORIGINS=http://localhost,http://127.0.0.1
POSTGRES_DB=django_db
POSTGRES_USER=django_user
POSTGRES_PASSWORD=senha_segura_do_banco
```

Para gerar uma `SECRET_KEY` aleatória, use:

```bash
python -c "import secrets; print(secrets.token_urlsafe(50))"
```

Ao publicar em um servidor, coloque o domínio ou IP dele em `ALLOWED_HOSTS` (ex.: `ALLOWED_HOSTS=meusite.com.br`) e a URL completa em `CSRF_TRUSTED_ORIGINS` (ex.: `CSRF_TRUSTED_ORIGINS=https://meusite.com.br`).

!!! danger "Importante!"
    Adicione o arquivo `.env` ao seu `.gitignore`. Nunca envie senhas e chaves secretas para o repositório Git!

## Configuração do Nginx (nginx/nginx.conf)

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
        server_name localhost;
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

## Configuração do Django (settings.py)

Modifique o arquivo `settings.py` do seu projeto Django:

```python
import os
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent

# Segurança
SECRET_KEY = os.environ.get('SECRET_KEY', 'chave-padrao-desenvolvimento')
DEBUG = os.environ.get('DEBUG', 'False') == 'True'
ALLOWED_HOSTS = os.environ.get('ALLOWED_HOSTS', 'localhost,127.0.0.1').split(',')
CSRF_TRUSTED_ORIGINS = os.environ.get(
    'CSRF_TRUSTED_ORIGINS', 'http://localhost,http://127.0.0.1'
).split(',')

# Banco de dados PostgreSQL
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('POSTGRES_DB', 'django_db'),
        'USER': os.environ.get('POSTGRES_USER', 'django_user'),
        'PASSWORD': os.environ.get('POSTGRES_PASSWORD', 'senha'),
        'HOST': os.environ.get('DB_HOST', 'db'),
        'PORT': os.environ.get('DB_PORT', '5432'),
    }
}

# Arquivos estáticos
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'

# Arquivos de mídia (uploads)
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'
```

Mantenha o restante do `settings.py` (`INSTALLED_APPS`, `MIDDLEWARE`, `TEMPLATES` etc.) como já está no seu projeto. Substitua apenas as configurações mostradas acima.

## .dockerignore

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

## Executando o Projeto

Com todos os arquivos criados, execute os seguintes comandos:

```bash
# Construir e iniciar os containers
docker compose up -d --build

# Verificar se os containers estão rodando
docker compose ps

# Ver logs dos containers
docker compose logs -f
```

Acesse http://localhost no navegador para ver sua aplicação.

## Criando Superusuário

Para criar um superusuário do Django:

```bash
docker compose exec web python manage.py createsuperuser
```

## Comandos Úteis

### Gerenciamento dos Containers

```bash
# Parar os containers
docker compose down

# Parar e remover os volumes (apaga dados do banco!)
docker compose down -v

# Reiniciar apenas um serviço
docker compose restart web

# Reconstruir um serviço específico
docker compose build web

# Ver status dos serviços
docker compose ps
```

### Executando Comandos no Container

```bash
# Acessar o shell do container web
docker compose exec web bash

# Executar migrations
docker compose exec web python manage.py migrate

# Criar nova app Django
docker compose exec web python manage.py startapp nome_app

# Acessar o shell do Django
docker compose exec web python manage.py shell

# Acessar o banco de dados
docker compose exec db psql -U django_user django_db
```

### Logs e Debug

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
docker compose exec -T db pg_dump -U django_user django_db > backup.sql
```

### Restaurar Backup

```bash
docker compose exec -T db psql -U django_user django_db < backup.sql
```

## Atualizando o Projeto

Após fazer alterações no código:

```bash
# Reconstruir a imagem e recriar os containers alterados
docker compose up -d --build
```

Não é preciso rodar `migrate` e `collectstatic` manualmente: o `entrypoint.sh` faz isso toda vez que o container `web` inicia.

## Solução de Problemas

### Container não inicia

Verifique os logs do container:

```bash
docker compose logs web
```

### Erro de conexão com banco de dados

1. Verifique se o container do banco está rodando e com status `healthy`: `docker compose ps`
2. Verifique as variáveis de ambiente no `.env`
3. Verifique os logs do banco: `docker compose logs db`

### Arquivos estáticos não carregam

1. Verifique se o `collectstatic` foi executado
2. Verifique as configurações de `STATIC_ROOT` e `STATIC_URL`
3. Verifique os volumes no `docker-compose.yml`

### Permissões de arquivo

Se encontrar erros de permissão, pode ser necessário ajustar as permissões:

```bash
docker compose exec web chmod -R 755 /app/staticfiles
docker compose exec web chmod -R 755 /app/media
```

## Considerações de Produção

Para uso em produção, considere:

- Configurar HTTPS com certificados SSL (Let's Encrypt)
- Usar senhas fortes e únicas
- Configurar firewall no servidor
- Implementar backup automatizado do banco de dados
- Configurar monitoramento e alertas
- Usar um registry privado para as imagens Docker
- Configurar health checks também nos containers `web` e `nginx`
- Ao usar HTTPS atrás do Nginx, adicionar `SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')` ao `settings.py`
