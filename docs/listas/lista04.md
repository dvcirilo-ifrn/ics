# Unidade 04

1. O que é conteinerização? Quais as principais diferenças entre máquinas virtuais e containers?
1. Qual a diferença entre imagem e container no Docker? Qual a função do Dockerfile nesse processo?
1. O que é o Docker Compose e qual problema ele resolve em relação ao uso de comandos `docker run` individuais?
1. No deploy Django com Docker Compose, qual a função de cada um dos serviços `db`, `web` e `nginx`?
1. Por que a diretiva `depends_on` sozinha não garante que o banco de dados esteja pronto? Como o `healthcheck` resolve esse problema?
1. Qual a diferença entre um bind mount e um volume nomeado? Por que o banco de dados utiliza um volume nomeado?
1. Por que os volumes de arquivos estáticos e de media são montados tanto no container `web` quanto no `nginx`?
1. Como os containers se comunicam dentro de uma rede do Docker Compose? Por que o Nginx pode usar `server web:8000` em vez de um endereço IP?
1. Como o Docker Compose utiliza o arquivo `.env`? Por que esse arquivo não deve ser adicionado ao repositório Git?
1. Qual a função do script `entrypoint.sh` no container Django? O que é necessário executar no servidor para atualizar o projeto após um `git pull`?
