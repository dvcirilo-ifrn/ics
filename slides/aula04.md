---
theme: default
size: 4:3
marp: true
paginate: true
_paginate: false
title: Aula 04: Formatação e Sistema de arquivos
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

**Aula 04**: Formatação e Sistema de Arquivos

---
# Formatação
- A formatação de discos é divida em três processos:
    - Formatação de baixo nível, ou formatação física;
    - Particionamento;
    - Formatação de alto nível, ou formatação lógica, onde são aplicados os sistemas de arquivos;

---
# Armazenamento em disco
- Discos magnéticos são grandes regiões de material onde é possível armazenar dados binários em formato magnético;
- É necessário organizar a escrita desses dados para possibilitar a leitura;
- Para tanto é feita a formatação de baixo nível, ou formatação física;
- Dentro do espaço vazio do disco são criadas estruturas de divisão do disco em trilhas e setores;

---
# Armazenamento em disco
- **Trilha**: Cada um dos círculos onde a cabeça de leitura/escrita pode atuar;
- **Setor**: Segmentos de uma trilha, armazenam quantidades fixas de dados: 512 bytes, 2048 bytes para CDs/DVDs e atualmente 4096 bytes (*Advanced Format*). Setores também armazenam informações de verificação dos dados;

---
# *Master Boot Record* (MBR)
- Espaço em disco responsável por armazenar informações de como as partições, contendo sistemas de arquivos, estão organizadas na mídia em questão.
- Além disso a MBR contém código executável com funções para inicialização do sistema operacional.
- Possui no mínimo uma tabela de partições e o código de chamada do sistema operacional;

---
# *Master Boot Record*
- Permite no máximo 4 partições, por limitações históricas (2 bits de endereçamento);
- A solução foi a criação do conceito de partições estendidas, que comportam até 255 partições lógicas (8 bits de endereçamento);
- Em teoria podemos ter 4 partições estendidas, cada uma com 255 partições lógicas;
- Armazena dados de tamanho e início/fim de partições em 32 bits, limitando o tamanho da partição a 2TB com setores de 512 bytes.

---
# *GUID Partition Table* (GPT)
- GUID: *Globally Unique Identifier*
- Utilizada atualmente em conjunto com a UEFI
- Resolve a questão das limitações da MBR

---
# MBR vs. GPT
![width:600px](../img/mbrgpt.png)

---
# Sistemas de arquivos
>O sistema responsável por armazenar, recuperar e atualizar um conjunto de arquivos. O termo se refere tanto as estruturas utilizadas para definir os arquivos, como os componentes de *software* que implementam suas camadas de abstração;

---
# Características
- O sistema de arquivos é responsável por organizar e gerenciar o armazenamento de arquivos e diretórios;
- Permite o acesso através de "nomes" e "extensões";
- Gerencia (aloca) o espaço utilizado no disco *cluster* e bloco);
- Distribui os dados de maneira otimizada;
- Permite o armazenamento de diversos atributos para cada arquivo ou diretório;
- Também é responsável por manter a integridade desses dados;
- *Journaling*;
- Fragmentação.

---
# FAT
- *File Allocation Table*;
- Um dos sistemas de arquivos mais simples e antigos;
- Leve e onipresente;
- Baseado em uma tabela de alocação de arquivos, que contém os endereços dos *clusters* de início e fim de cada arquivo;
- Define o *cluster* ou bloco, que é a unidade mínima de dado que pode ser escrita pelo sistema;
- Um *cluster* pode conter mais de um setor;
- Arquivos são divididos em clusters;
- *Slack space*;

---
# FAT
- FAT16:
    - Cada *cluster* pode conter até 64 setores;
    - 16 bits para endereçamento de *clusters*;
    - Máximo de 2GB por partição;
- FAT32:
    - 32 bits de endereçamento;
    - Permite o uso de *clusters* de 4KB, reduzindo o desperdício de espaço, mas apenas em partições de até 8GB;
    - Limita o tamanho dos arquivos a 4GB;

---
# NTFS
- *New Technology File System*
- 64 bits de endereçamento;
- Tamanho variável de *cluster*;
- Sistema com *journaling*;
- Nomes armazenados em Unicode;
- Possibilidade de criptografia;
- Está sendo substituído no Windows 11 pelo ReFS, também proprietário.

---
# EXT3/4
- *Extended File System*;
- Sistema nativo do Linux;
- Apresenta todas as vantagens do NTFS;
- Menor fragmentação do que o NTFS;
- Permite mais caracteres em nomes de arquivos;

---
# Estrutura de diretórios no Windows
- O Windows usa letras para definir as partições e discos;
- As letras A: e B: são reservadas pra os disquetes;
- O disco principal é o C:
- [Windows Directory Structure](https://en.wikipedia.org/wiki/Directory_structure)

---
# Estrutura de diretórios no Linux
- As partições no Linux são *transparentes* para o usuário;
- Ex. uma pasta qualquer pode estar em outra partição;
- Os discos físicos no Linux são definidos em `/dev/sda`, `/dev/sdb`, `/dev/sdc`, etc.
- As partições são números. Ex. `/dev/sda1`, `/dev/sda2`.
- Tais partições podem ser *montadas* em qualquer diretório, virando uma espécie de atalho.
- [Filesystem Hierarchy Standard](https://pt.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)
---
# <!--fit--> Dúvidas? 🤔
