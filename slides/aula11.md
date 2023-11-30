---
marp: true
theme: default
paginate: true
_paginate: false
size: 4:3
title: Aula 11: Banco de Dados MySQL/mariadb
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

**Aula 11**: Banco de Dados MySQL/MariaDB

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
```sh
$ sudo systemctl status mariadb
```
---
# Padrão de segurança
```sh
$ sudo mysql_secure_installation
```
- Switch to unix_socket authentication [Y/n] - `n` para pular.
- Set root password? [Y/n] - Digite `y` e aperte `Enter` para criar uma senha forte de `root` para seu BD. 
- Remove anonymous users? [Y/n] - Digite `y` e `Enter`.
- Disallow root login remotely? [Y/n] - Digite `y` e `Enter`.
- Remove test database and access to it? [Y/n] - Digite `y` e `Enter`.
- Reload privilege tables now? [Y/n] - Digite `y` e `Enter`.
---
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
---
# phpMyAdmin
- Cliente web para MySQL/MariaDB;
- Funciona em PHP, que deve estar ativado no servidor web (Apache/Nginx);
- Instale os pacotes do PHP:
```sh
$ sudo apt install php php-cgi php-mysqli php-pear php-mbstring libapache2-mod-php php-common php-phpseclib php-mysql
```
- Verifique a instalação:
```sh
$ php --version
```

---
# phpMyAdmin
- Baixe o phpMyAdmin do site:
```sh
$ wget https://www.phpmyadmin.net/downloads/phpMyAdmin-latest-all-languages.tar.gz
```
- Crie uma pasta na raiz do servidor web (`/var/www/html/`)
```sh
$ sudo mkdir /var/www/html/phpMyAdmin
```
- Descompacte a pasta para o servidor web:
```sh
$ sudo tar xvf phpMyAdmin-latest-all-languages.tar.gz --strip-components=1 -C /var/www/html/phpMyAdmin
```

---
# phpMyAdmin
- Copie a configuração padrão:
```sh
$ sudo cp /var/www/html/phpMyAdmin/config.sample.inc.php /var/www/html/phpMyAdmin/config.inc.php
```

- Edite a configuração para modificar a senha:
```sh
sudo nano /var/www/html/phpMyAdmin/config.inc.php
```

- Mude de:
```sh
$cfg['blowfish_secret'] = '';
```
- Para:
```sh
$cfg['blowfish_secret'] = 'Sua-Nova-Senha-Complexa';
```
---
# phpMyAdmin
- Mude as permissões do arquivo:
```sh
sudo chmod 660 /var/www/html/phpMyAdmin/config.inc.php
```
- Mude o dono para o usuário do Apache:
```sh
sudo chown -R www-data:www-data /var/www/html/phpMyAdmin
```
- Reinicie o Apache2:
```sh
sudo systemctl restart apache2
```
- Acesse em http://localhost:8888/phpMyAdmin

---
# <!--fit--> Dúvidas? 🤔
