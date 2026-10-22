---
marp: true
theme: default
paginate: true
_paginate: false
size: 4:3
title: Aula 12: Banco de Dados MySQL/mariadb
author: Diego Cirilo
---
<style>
img, table {
  display: block;
  margin: 0 auto;
}
</style>

# <!-- fit --> Instalação e Configuração de Servidores

### Prof. Diego Cirilo

**Aula 12**: Banco de Dados MySQL/MariaDB

---
# MySQL/MariaDB

- Sistema de gerenciamento de Banco de Dados Relacional (RDBMS);
- Criado em 1995 por David Axmark e Michael Widenius, na Suécia;
- My (filha de Widenius) + SQL;
- Software Livre, porém desenvolvido atualmente pela Oracle;
- Widenius criou um fork em 2010 chamado MariaDB, nome de sua filha mais nova.

---
# Instalação

- O MySQL não faz parte dos repositórios do Debian, MariaDB sim.
- Instalação:
```sh
$ sudo apt install mariadb-server mariadb-client
```
- Verifique o funcionamento:
```sh
$ sudo systemctl status mariadb
```
---
<style scoped>section { font-size: 22px; }</style>
# Configuração
- Para acessar o BD:
```sh
$ sudo mysql -u root
```
- Crie um usuário administrador:
```sql
CREATE USER 'admin'@localhost IDENTIFIED BY 'senhaadmin';
```
- Dê permissões totais:
```sql
GRANT ALL PRIVILEGES ON *.* TO 'admin'@localhost IDENTIFIED BY 'senhaadmin';
```
- Atualize as permissões:
```sql
FLUSH PRIVILEGES;
```
- Saia do MariaDB: `quit`

---
# Criando usuário específico para um banco

- Normalmente os usuários do BD só tem acesso aos seus bancos;
- Além de mais seguro, evita erros;
- Acesse o BD como `root` ou `admin`;
- Execute os seguintes comandos SQL:
```sql
CREATE DATABASE nome_do_banco CHARACTER SET 'utf8';
CREATE USER nome_do_user;
GRANT ALL ON nome_do_banco.* TO 'nome_do_user'@'localhost' IDENTIFIED BY 'senha_do_user';
quit
```

---
# Acesso remoto

- Podemos acessar o BD diretamente no terminal:
    - Acesse o servidor via SSH;
        - `$ mysql -u admin`
        - Onde `admin` é o nome de usuário.
    - Esse terminal permite a execução direta de comandos SQL;
- Podemos também usar ferramentas gráficas como o MySQL Workbench ou phpMyAdmin.

---
<style scoped>section { font-size: 24px; }</style>
# Acesso remoto com MySQL Workbench

- Na janela de "nova conexão" escolhemos o "Connection Method" *Standard TCP/IP over SSH*
    - *SSH Hostname*: IP do servidor remoto
    - *SSH Username*: usuário do servidor remoto
    - *SSH Password*: senha do servidor remoto (apenas quando não usarmos chave);
    - *SSH Key File*: chave privada. Procure as chaves que criou localmente e configurou no servidor remoto;
    - *MySQL Hostname* e *MySQL Server Port*: pode deixar o padrão;
    - *Username* e *Password*: dados do usuário do MySQL. Ex. `admin` e `senhaadmin`;
- Não esqueça de definir um *Connection Name* para poder salvar.

---
# phpMyAdmin

- Cliente web para MySQL/MariaDB;
- Funciona em PHP, que deve estar ativado no servidor web (Apache/Nginx);
- O Debian já possui o pacote do phpMyAdmin.
- Também é possível instalar manualmente, baixando os arquivos no site oficial;
- [phpMyAdmin](https://www.phpmyadmin.net/)

---
# phpMyAdmin no nginx

- Verifique se o *nginx* está instalado e funcionando!
- Instale o pacote do *phpMyAdmin* e as dependências do *php*

```sh
sudo apt install phpmyadmin php-fpm php-mysql
```
- O instalador perguntará se deseja configurar para o `Apache` ou para o `lighttpd`, como não usaremos nenhum dos dois, **não marque nenhuma opção** e selecione o *OK*.
- Na sequência perguntará se deve ser feita a configuração (*yes*) e pedirá uma senha, que pode ficar em branco. O instalador gera uma senha forte, nesse caso.

---
# phpMyAdmin no nginx
- Verifique se o arquivo de configurações do site *default* do nginx está com o PHP habilitado:
```sh
sudo nano /etc/nginx/sites-available/default
```
- Adicione `index.php` à lista de arquivos padrão:
```
# Add index.php to the list if you are using PHP
index index.html index.htm index.nginx-debian.html index.php;
```

---
# phpMyAdmin no nginx
- As seguintes linhas devem estar sem comentário:

```conf
# pass PHP scripts to FastCGI server
#
location ~ \.php$ {
        include snippets/fastcgi-php.conf;

        # With php-fpm (or other unix sockets):
        fastcgi_pass unix:/run/php/php8.2-fpm.sock;
        # With php-cgi (or other tcp sockets):
#       fastcgi_pass 127.0.0.1:9000;
} # LEMBRE DESSE!!
```

> **ATENÇÃO**: ajuste a versão do PHP na linha do php8.2-fpm.sock, caso sua versão não seja a 8.2.

---
# phpMyAdmin no nginx

- Crie um link simbólico para o phpMyAdmin na pasta `/var/www/html`
```sh
sudo ln -s /usr/share/phpmyadmin /var/www/html/phpmyadmin
```
- Reinicie o *nginx* e verifique o funcionamento em http://endereco-do-servidor/phpmyadmin

---
# <!--fit--> Dúvidas? 🤔
