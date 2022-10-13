---
marp: true
theme: default
size: 4:3
paginate: true
_paginate: false
title: Aula 01: Configuração do Debian
author: Diego Cirilo

---
<style>
img {
  display: block;
  margin: 0 auto;
}
</style>

# <!-- fit --> Instalação e Configuração de Servidores

### Prof. Diego Cirilo

**Aula 01**: Configuração de servidor Debian

---
# Configurações iniciais

- Acesse a VM com seu usuário/senha (não use o root)
- Vire *super user* com `su -` (entenda o `-` lendo o manual: `man su`)
- Atualize o sistema e instale o pacote `sudo`

```shell
$ su - 

# apt update
# apt full-upgrade
# apt install sudo
```

---
# Configurando o `sudo`
- Provavelmente seu usuário não ter permissão de usar o `sudo`
- Saia do login de *root* e teste! Ex. `sudo apt update`
- Para configurar as permissões de `sudo` usamos o `visudo`

```shell
$ su -
# visudo
```

---
# /etc/sudoers

```bash
#
# This file MUST be edited with the 'visudo' command as root.
#
# Please consider adding local content in /etc/sudoers.d/ instead of
# directly modifying this file.
#
# See the man page for details on how to write a sudoers file.
#
Defaults	env_reset
Defaults	mail_badpass
Defaults	secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

# Host alias specification

# User alias specification

# Cmnd alias specification

# User privilege specification
root	ALL=(ALL:ALL) ALL

# Allow members of group sudo to execute any command
%sudo	ALL=(ALL:ALL) ALL

# See sudoers(5) for more information on "@include" directives:

@includedir /etc/sudoers.d
```

---
# `/etc/sudoers`

- **root** ALL=(ALL:ALL) ALL - Usuário ou grupo, se iniciar com `%`
- root **ALL**=(ALL:ALL) ALL - Em que hosts pode executar
- root ALL=(**ALL**:ALL) ALL - Como qual usuário
- root ALL=(ALL:**ALL**) ALL - Com qual grupo
- root ALL=(ALL:ALL) **ALL** - Quais comandos

---

# Entrando no grupo *sudo*

- Podemos criar permissões para nosso usuário ou entrar no grupo *sudo*
- Para entrar no grupo:
```shell
# usermod -aG sudo usuario
# exit
$ exit
```
- E fazemos login novamente para as mudanças terem efeito

---
# Verificando
- Verifique se faz parte do grupo com:
```shell
$ groups
```
- E teste:
```shell
$ sudo apt update
```
---
# <!--fit--> Dúvidas? 🤔
