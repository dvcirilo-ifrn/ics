# Unidade 04

1. O que significa fazer o "deploy" de uma aplicação? Qual a diferença fundamental entre os ambientes de desenvolvimento e produção?
1. Por que não devemos reutilizar o ambiente virtual (venv) entre diferentes máquinas? Como devemos proceder para garantir que as dependências sejam instaladas corretamente em diferentes ambientes?
1. Explique a importância de configurar `DEBUG=False` em produção. Quais são os riscos de manter `DEBUG=True` em um servidor de produção?
1. Por que é importante nunca adicionar o `SECRET_KEY` de produção ao repositório Git? Como devemos gerenciar informações sensíveis como senhas e chaves secretas?
1. O que é o Gunicorn e qual sua função no deploy de aplicações Django? Por que não usamos o servidor de desenvolvimento do Django (runserver) em produção?
1. Explique o funcionamento do Gunicorn como daemon no systemd. Qual a diferença entre os arquivos `.socket` e `.service`?
1. Qual a função do Nginx no contexto do deploy de uma aplicação Django? Por que utilizamos o Nginx como proxy reverso em vez de acessar o Gunicorn diretamente?
1. Como o Nginx serve arquivos estáticos (CSS, JavaScript, imagens) de forma diferente do conteúdo dinâmico da aplicação Django?
1. Explique o processo de coleta de arquivos estáticos com o comando `collectstatic`. Por que esse passo é necessário?
1. Ao configurar múltiplos projetos Django em um mesmo servidor, quais estratégias podem ser utilizadas? Quais os cuidados necessários ao usar subdiretórios?
1. Descreva o processo completo de atualização de um projeto Django em produção após fazer um `git pull`. Quais comandos devem ser executados e em que ordem?
1. O que é conteinerização? Como essa tecnologia difere da virtualização tradicional?
1. Compare máquinas virtuais e containers em relação aos seguintes aspectos:
    1. Uso de memória
    1. Tempo de inicialização
    1. Compartilhamento de recursos
    1. Nível de isolamento
    1. Portabilidade
1. Explique os seguintes conceitos fundamentais do Docker:
    1. Imagem
    1. Container
    1. Volume
    1. Network
    1. Registry
1. O que é um Dockerfile? Explique a função das seguintes instruções:
    1. `FROM`
    1. `WORKDIR`
    1. `COPY` e `ADD`
    1. `RUN`
    1. `CMD` e `ENTRYPOINT`
    1. `EXPOSE`
1. Qual a diferença entre construir uma imagem Docker e executar um container? Apresente os comandos para ambas as operações.
1. Explique as seguintes opções do comando `docker run`:
    1. `-d`
    1. `-p 8000:8000`
    1. `-v $(pwd)/data:/app/data`
    1. `-e DEBUG=False`
    1. `--name`
    1. `--rm`
1. O que é o Docker Hub? Como podemos baixar uma imagem oficial do Docker Hub e como podemos enviar nossas próprias imagens?
1. O que é o Docker Compose e qual problema ele resolve? Em que situações é mais vantajoso usar Docker Compose em vez de comandos `docker run` individuais?
1. Explique a estrutura básica de um arquivo `docker-compose.yml`. O que são "services", "volumes" e "networks" nesse contexto?
1. Quais as principais diferenças entre usar `docker-compose up` e `docker-compose up -d`? Como visualizar os logs dos serviços após iniciá-los em background?
1. No contexto do tutorial de deploy Django com Docker, explique a função de cada um dos seguintes serviços:
    1. `db` (PostgreSQL)
    1. `web` (Django + Gunicorn)
    1. `nginx`
1. Por que utilizamos um script `entrypoint.sh` no container Django? O que esse script faz antes de iniciar o Gunicorn?
1. Como o Docker Compose gerencia a ordem de inicialização dos containers usando a diretiva `depends_on`? Isso garante que o banco de dados esteja completamente pronto antes do Django tentar se conectar?
1. Explique como funcionam os volumes nomeados no Docker Compose (ex: `postgres_data`, `static_volume`, `media_volume`). Por que eles são importantes para persistência de dados?
1. Como o Nginx se comunica com o Gunicorn dentro da rede Docker? Por que podemos usar `server web:8000` em vez de um endereço IP?
1. Compare o deploy tradicional de Django (aula 13) com o deploy usando Docker (aula 14). Quais as vantagens e desvantagens de cada abordagem?
1. Qual a função do arquivo `.dockerignore`? Por que devemos excluir arquivos como `.git`, `venv`, `.env` e `__pycache__` da imagem Docker?
1. Quais comandos Docker/Docker Compose você usaria para:
    1. Ver os logs do serviço web em tempo real
    1. Executar as migrations do Django
    1. Criar um superusuário
    1. Acessar o shell bash do container web
    1. Fazer backup do banco de dados PostgreSQL
1. Cite pelo menos 5 boas práticas ao trabalhar com Docker em produção.
1. Quais tecnologias de orquestração de containers você pode usar além do Docker Compose para ambientes de produção em larga escala?
