---
marp: true
theme: default
paginate: true
_paginate: false
size: 4:3
title: Aula 08: Configurações e acesso remoto
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

**Aula 08**: Configurações e acesso remoto

---

# Configuração de rede do Virtualbox
- Rede NAT
    - *Network Address Translation*
    - *Port-forwarding*
    - Ex. Redirecionar a porta 8888 no *host* para a porta 80 no *guest*.
- *Bridge*
    - A VM é vista como uma máquina real na rede.
    - O IP e as configurações passam a ser dadas pelo roteador.

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
- SFTP/SSH FTP

---
# Filezilla

- Host: `sftp://localhost`
- Port: `2222`
- Usuário/Senha

---
# Editores de texto no terminal

- nano/pico
    - Mais simples, comandos para salvar, sair, etc, usando CTRL.
- vi/vim
    - Pode ser usado como IDE completa, extremamente poderoso
    - Opera no conceito de modos: comando, inserção e visual

---
# Dicas no Vi/Vim
- Para sair do modo de inserção use o `ESC`
- Para entrar no modo de inserção (escrever) use `i`
- Para sair do programa sem salvar: `:q!`
- Para salvar e sair: `:wq`
- [Mais comandos](https://vim.rtorr.com/lang/pt_br)

---
# Systemd

- Gerenciamento do sistema por meio de *units*:
    - `service`, `mount`, `socket`, `device`
- Sintaxe:
    - `# systemctl <operaçao> <unit>`
    - Ex. `$ sudo systemctl status apache2.service`
---
# Systemd
<style scoped>
table {
  font-size: 18px;
}
</style>

| Operação | Função |
|---|---|
|`start`| Inicializa *unit* |
|`stop`| Parar |
|`restart`| Reiniciar |
|`reload`| Recarrega configurações |
|`status`| Status |
|`enable`| Habilitar |
|`disable`| Desabilitar |
|`is-enabled`| Verifica se está habilitado |

---
# Outras funções do Systemd
- Desligar: `# systemctl poweroff`
- Reiniciar: `# systemctl reboot`
- Suspender: `# systemctl suspend`
- Hibernar: `# systemctl hibernate`

---
# Tarefa
- Crie um arquivo de texto no Windows e envie para o servidor:
    - Usando scp;
    - Usando Filezilla

- Acesse o servidor utilizando ssh

---
# <!--fit--> Dúvidas? 🤔
