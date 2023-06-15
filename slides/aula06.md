---
theme: default
size: 4:3
marp: true
paginate: true
_paginate: false
title: Aula 06: Comandos Linux
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

**Aula 06**: Comandos Linux

---
# Informações gerais

- No terminal, o espaço onde digitamos comandos é chamado de *prompt* de comando.
- No *prompt* normalmente é indicado o usuário, o nome da máquina e o diretório atual.
- `diego@debian:$`
- Usuários comuns são representados por `$` e o usuário *root* como `#`.
- Nada aparece quando digitamos senhas.
- Nada aparece quando o comando dá certo (salvo exceções).

---
# Estrutura de um comando

- Letras maiúsculas e minúsculas importam!! (case-sensitive)
- As opções podem ser apresentadas por extenso com `--` ou abreviadas com `-`

```sh
$ comando --opcoes -o argumentos
```
---
# Exemplos 
```sh
$ ls
$ mv arquivo ../pasta-acima/outroarquivo
# chmod +x arquivo
$ wget --help
```

---
# Comandos básicos
- `ls` lista o conteúdo do diretório. Opções úteis:
    - `ls -l` lista detalhes dos arquivos
    - `ls -a` lista todos os arquivos, inclusive "ocultos" (iniciam com `.`)
    - `ls -lah` lista detalhes de todos os arquivos e mostra valores legíveis (*human*)
- `pwd` indica o diretório atual
- `whoami` indica o usuário atual
- `cd destino` muda do diretório atual para destino (entra e sai de pastas)
    - `..` sobe um nível no diretório
    - `~` vai para o diretório do usuário (*home*)
---
# Linux FHS
- Linux *Filesystem Hierarchy Standard*
- Define os diretórios padrão.
- Ex.
    - A raiz do sistema é o `/`
    - Os arquivos do usuário ficam em `/home/usuario`
    - Arquivos de configuração em `/etc`
- Lista completa: [Linux FHS](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)

---
# Endereçamento absoluto e relativo
- Absoluto: caminho completo a partir da raiz, logo SEMPRE começa com `/`
    - `/home/diego/ifrn/meu-arquivo`
- Relativo: depende do diretório atual.
    - É possível referenciar níveis acima com `..`
    - Por padrão os níveis são abaixo.
    - Ex. se eu estiver na pasta `/home/diego` e quiser entrar em `ifrn/meu-diretorio` faço:
    - `cd ifrn/meu-diretorio`

---
# Criação de diretórios
- `mkdir nome`
- Atenção para as permissões!

---
# <!--fit--> Dúvidas? 🤔
