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
- `diego@debian:~$`
- Usuários comuns são representados por `$` e o usuário *root* como `#`.
- Nada aparece quando digitamos senhas.
- Nada aparece quando o comando dá certo (salvo exceções).
- Use a tecla TAB no teclado para completar os comandos e ver sugestões.
- Use as setas para cima e para baixo para ver o histórico de comandos.

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
- Lista completa: [Linux FHS](https://pt.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)

---
# Endereçamento absoluto e relativo
- Absoluto: caminho completo a partir da raiz, logo **SEMPRE** começa com `/`
    - `/home/diego/ifrn/meu-arquivo`
- Relativo: depende do diretório atual.
    - É possível referenciar níveis acima com `..`
    - Por padrão os níveis são abaixo.
    - Ex. se eu estiver na pasta `/home/diego` e quiser entrar em `ifrn/meu-diretorio` faço:
    - `cd ifrn/meu-diretorio`

---
# Criação de diretórios e arquivos
- `mkdir nome` - Cria diretório
- `touch nome` - Cria arquivo vazio
- Normalmente não são permitidos acentos e espaços
    - É necessário escrever entre aspas. Ex.
    - `touch "meu arquivo absurdo"`
    - De toda forma não é recomendável!

---
# Removendo arquivos e diretórios
- `rm nome`
- Opções:
    - `rm -r diretorio` - Recursivo
    - `rm -rf diretorio` - Recursivo e "forçado"
- Coringas (wildcards) - **CUIDADO**
    - `rm *.pdf` - remove todos os pdfs de um diretório
    - `rm tes*`  - remove todos os arquivos que começam com `tes`
    - `rm -rf *` - remove todos os arquivos de um diretório

---
# Exemplo
- Na sua *home*, crie a seguinte estrutura de diretórios:
    - `filmes`
        - `nacionais`
            - `comedia`
            - `drama`
        - `internacionais`
            - `dublados`
            - `legendados`
    - `series`
        - `pra-assistir`
        - `assistidas`
- Crie pelo menos um arquivo em cada diretório final (os mais internos).
- Na sua *home* (use `cd ~` para voltar para home), faça `ls -lR` para listar todas os diretórios recursivamente.
- Apague tudo :)

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
# <!--fit--> Dúvidas? 🤔
