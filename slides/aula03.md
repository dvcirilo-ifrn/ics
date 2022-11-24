---
marp: true
theme: default
paginate: true
_paginate: false
size: 4:3
title: Aula 03: Acesso remoto ao servidor
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

**Aula 03**: Acesso remoto ao servidor

---
# Acesso SSH
- Secure Shell

- Instalar o OpenSSH Server:

`$ sudo apt install openssh-server`

- Da sua máquina acessar o servidor com:
`$ ssh <usuario>@<servidor> -p<porta>`

- No VirtualBox redirecionar a porta 2222 (exemplo) para a porta 22 no servidor

---
# Transferência de arquivos
- scp - Secure Copy
`$ scp -Pporta origem destino`
- Ex. `scp -P2223 arquivo.html diego@localhost:arquivo.html`
- SFTP/SSH FTP - Filezilla

---
# <!--fit--> Dúvidas? 🤔

