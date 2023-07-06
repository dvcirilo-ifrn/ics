---
theme: default
size: 4:3
marp: true
paginate: true
_paginate: false
title: Aula 07: Usuários, Grupos e Permissões
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

**Aula 07**: Usuários, Grupos e Permissões

---
# Usuários e Grupos
- Funcionalidade para gerenciamento de permissões de acesso
- Cada usuário pode pertencer a um ou mais grupos
- Quais grupos eu faço parte?
    - `groups`
- Os usuários ficam listados no arquivo `/etc/passwd`
- Os grupos ficam listados no arquivo `/etc/group`

---
# Permissões e propriedade
- Arquivos pertencem a um usuário e a um grupo
- No Linux/Unix *everything is a file*
- Usuários e grupos tem níveis de permissão para um arquivo:
    - `r` *read*
    - `w` *write*
    - `x` *execute*
- A ordem no `ls -l` é: dono, grupo e outros.

---
# Exemplo
![width:800px](../img/lsout.png)

---
<style scoped>section { font-size: 24px; }</style>
# Configurando permissões
- `chmod permissoes arquivos`
- As permissões podem ser no formato binário ou com letras.
- Binário
    - Os bits na ordem `rwx` convertidos para decimal.
    - Ex. `101` é 5, `100` é 4, etc.
    - Usamos um dígito para cada usuário, na ordem dono, grupo e outros.
    - Ex. "644", "777"
- Letras
    - Usamos `+` para adicionar e `-` para remover
    - `u` para o dono, `g` para o grupo e `o` para os outros. `a` para todos.
    - Ex. `a+x`, `go-x`, etc.

---
# Exercício
- Crie um diretório na sua `home` chamado `tarefa`
- Dentro desse diretório, crie 3 arquivos com as seguintes permissões:
    - `arq1`: o dono pode ler, escrever e executar, o grupo pode ler e executar, outros podem ler.
    - `arq2`: o dono pode ler e escrever, o grupo pode ler, outros não podem nada.
    - `arq3`: o dono pode ler e escrever, o grupo pode ler e escrever, outros podem ler e escrever.
- Verifique o resultado com `ls -l`

---
# Exercício
- Altere as permissões dos arquivos criados no exercício anterior:
    - `arq1`: remova a permissão de execução do dono e grupo.
    - `arq2`: adicione a permissão de escrita e execução para todos.
    - `arq3`: ninguém pode mais escrever.
- Verifique o resultado com `ls -l`

---
# <!--fit--> Dúvidas? 🤔
