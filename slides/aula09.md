---
marp: true
theme: default
paginate: true
_paginate: false
size: 4:3
title: Aula 09: Instalando o servidor web Apache2
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

**Aula 09**: Instalando o servidor web Apache2

---
# Servidores Web mais utilizados

![height:550px](../img/servers.png)

---
# Instalação do Apache2

```shell
$ sudo apt update && sudo apt full-upgrade
$ sudo apt install apache2
```
---
# Conceitos básicos do Apache2

- Sites - VirtualHosts
    - Servidores "virtuais", permite servir mais de um site em um mesmo servidor;
- Módulos
    - Extensões para funções além do HTTP estático, ex. PHP e Python.
    - Instala com `apt install libapache2-mod-<nome>`
- Configurações

---

# Configuração do Apache2
- Arquivo base de configuração `/etc/apache2/apache2.conf`
- Conceito de *available* e *enabled*
    - `sites-available` -> `sites-enabled`
    - `conf-available` -> `conf-enabled`
    - `mods-available` -> `mods-enabled`
- *Symlinks*
    - `ln -s <origem> <link>`

---
# Configuração do Apache2

| Comando | Função |
|---|---|
| `a2ensite` | Habilita site |
| `a2dissite` | Desabilita site |
| `a2enconf` | Habilita configuração |
| `a2disconf` | Desabilita configuração |
| `a2enmod` | Habilita módulo |
| `a2dismod` | Desabilita módulo |

---
<style scoped>section { font-size: 22px; }</style>
# Sites/VirtualHosts
```conf
<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
</VirtualHost>
```

---
# DocumentRoot

- Caminho raiz do site
- Todos os arquivos aparecem na URL a partir do *DocumentRoot*
- Ex. www.meubelosite.com/fotos/index.html

---
# Tarefa
- Encontre o index.html padrão do Apache
- Crie uma cópia para preserva-lo (`cp <origem> <destino>`)
- Crie um novo index.html (pode usar CSS/JS)
- Acesse o site a partir da sua máquina Windows para verificar o funcionamento.

---
# Permissões de acesso
- O Apache roda como o usuário `www-data`;
- Esse usuário deve ter acesso de leitura ao diretório do site;
- É possível configurar com o `chmod 755 <diretorio>`.

---
<style scoped>section { font-size: 26px; }</style>
# Tarefa
- Crie um novo VirtualHost que aponte para uma pasta na sua *home*
- Dica: copie as configurações do *default* e modifique o arquivo.
- Adicione ao seu sites-available/meusite.conf:
```
<Directory /home/seuusuario/suapasta>
        AllowOverride All
        Options -Indexes
        Require all granted
</Directory>
```
- Configure as permissões de acesso para sua pasta
- Crie um site nessa pasta e verifique o acesso a partir do Windows.

---
# <!--fit--> Dúvidas? 🤔
