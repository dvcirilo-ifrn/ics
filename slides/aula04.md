---
marp: true
theme: default
paginate: true
_paginate: false
size: 4:3
title: Aula 04: Servidor Nginx
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

**Aula 04**: Servidor Web Nginx

---
# Nginx

- *engine x*/*enjin eks*
- Criado em 2004 por Igor Sysoev;
- Desenvolvido com o objetivo de ser mais leve que o Apache;
- Servidor mais utilizado atualmente (2022);

---
# Instalação

- `$ sudo apt install nginx`
- Usa o systemctl para controle como o Apache;
- Arquivos de configuração em `/etc/nginx` e site padrão em `/var/www/html`
- Mesmo padrão de `sites-available` e `sites-enabled`.


---
# Server Blocks (VirtualHosts)

- Possuem função equivalente ao VirtualHosts do Apache;
- São habilitados com links simbólicos (`ln -s <origem> <destino>`)
- São desabilitados removendo o link (`rm <arquivo>`)

---
# Tarefa

- Crie um novo site em `sites-available` copiando o `default` para `meusite`
- Modifique a localização do site na diretiva `root` para um site na sua *home*.
- Desabilite o site `default` (remova o link em `sites-enabled`)
- Habilite `meusite` criando link simbólico em `sites-enabled`
`$ sudo ln -s /etc/nginx/sites-available/meusite /etc/nginx/sites-enabled/meusite`
- Reinicie o Nginx
- Verifique o funcionamento no navegador.

---
# <!--fit--> Dúvidas? 🤔
