# Unidade 03

1. O que é o MariaDB/MySQL? Quais são as principais diferenças entre eles e por que o MariaDB é frequentemente utilizado em distribuições Linux como Debian?
1. Por que é importante criar um usuário administrador separado (como admin) em vez de usar o usuário root diretamente no MySQL/MariaDB?
1. Descreva o processo de criação de um banco de dados e de um usuário com permissões específicas para esse banco. Como essa prática aumenta a segurança?
1. Como acessar o terminal do MySQL como usuário root? Quais cuidados devem ser tomados ao acessar diretamente com root?
1. Qual a função do phpMyAdmin? Quais são as vantagens e riscos de utilizá-lo para administrar bancos de dados em servidores web?
1. Como instalar e configurar o phpMyAdmin com o Nginx e PHP-FPM? O que poderia acontecer se a configuração do PHP não fosse feita corretamente?
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
