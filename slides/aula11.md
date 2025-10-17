---
marp: true
theme: default
paginate: true
_paginate: false
size: 4:3
title: Aula 11: Segurança no acesso remoto
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

**Aula 11**: Segurança no acesso remoto

---
# SSH

- SSH (Secure Shell): protocolo de acesso remoto criptografado
- Permite: 
    - executar comandos
    - transferir arquivos
    - administrar servidores.  
- SSH substituiu protocolos inseguros como Telnet e rlogin.  

---
# Funcionamento do SSH

1. O cliente inicia a conexão  
2. O servidor se autentica enviando sua chave pública  
3. O cliente valida a identidade do servidor  
4. A comunicação é criptografada de ponta a ponta  

---
# Métodos de Autenticação

- Senhas: o usuário informa login e senha a cada conexão  
- Chaves SSH: o usuário usa um par de chaves (pública e privada) para autenticação automática e mais segura  

---
# Chave Pública e Privada

- As chaves SSH são baseadas em criptografia assimétrica;
    - Só funcionam juntas como um par matematicamente relacionado.  
    - Mesma lógica de vários sistemas de autenticação atuais.
- Chave Privada: permanece no cliente; deve ser protegida e nunca compartilhada  
- Chave Pública: é enviada ao servidor; usada para verificar a autenticidade do cliente  

---
# Processo de Autenticação com Chaves

1. O servidor armazena a chave pública do usuário autorizado  
2. O cliente inicia a conexão e apresenta sua chave pública  
3. O servidor envia um desafio criptografado com essa chave  
4. O cliente responde usando sua chave privada  
5. Se a resposta for válida, o acesso é concedido  

---
# Vantagens

- Elimina o uso de senhas, evitando ataques de força bruta  
- Permite automação segura de tarefas e deploys  
- Oferece autenticação forte baseada em criptografia  
- Facilita o uso de agentes de autenticação (`ssh-agent`)  
- Pode ser protegida por uma senha local (*passphrase*)  

---
# Geração de Chaves SSH

- `-t` tipo de criptografia, atualmente o ed25519 é recomendável. 
- `-C` comentário, facilita na identificação das chaves.

```
ssh-keygen -t ed25519 -C "seu_email@exemplo.com"
```

Por padrão o comando gera dois arquivos em `~/.ssh/`:

- `id_ed25519` (chave privada)  
- `id_ed25519.pub` (chave pública)  

---
# Adicionando a Chave ao Servidor

Para autorizar o acesso, copie a chave pública para o servidor:

```
ssh-copy-id usuario@servidor
```

Ou manualmente:

- Copie o conteúdo de `id_ed25519.pub`  
- Adicione ao arquivo `~/.ssh/authorized_keys` no servidor  

> Em plataformas de *cloud* é necessário usar as ferramentas da própria platorma.

---
# Adicionando chaves ao Google Cloud
- Acesse o Compute Engine (https://console.cloud.google.com/compute)
- No menu do lado esquerdo acesse *Configurações/Metadados*
- Acesse a aba *Chaves SSH* e use a opção *+ Adicionar item*
- Cole sua chave pública no campo que vai aparecer e salve.

---
# Protegendo a Chave Privada

- Mantenha a chave privada apenas no seu computador  
- Use permissões restritas:

```
chmod 600 ~/.ssh/id_ed25519
```

- Use uma passphrase para proteger o arquivo  
- **Nunca** envie a chave privada por e-mail ou mensagens  

---
# Agente SSH

O `ssh-agent` armazena chaves privadas na memória, evitando digitar a senha repetidamente.

Iniciar e adicionar a chave:

```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

---
# Desabilitando o Acesso por Senha

- É possível desativar o acesso por senha.
- No servidor, edite o arquivo `/etc/ssh/sshd_config`:

```
PasswordAuthentication no
PubkeyAuthentication yes
```

- Reinicie o serviço SSH:

```
sudo systemctl restart ssh
```

> ATENÇÃO: só é possível acessar com as chaves, e caso sejam perdidas, não é mais possível acessar o servidor remotamente!

---
# Boas práticas

- Identificar e documentar onde cada chave é usada  
- Utilizar nomes descritivos ao criar chaves  
- Revogar chaves antigas ou comprometidas  
- Limitar o acesso de cada chave a servidores específicos  
- Evitar chaves compartilhadas entre usuários  

---
# Reuso de Chaves SSH

- Facilita o acesso, porém aumenta o impacto de um possível vazamento  
- Se a chave for comprometida, todos os servidores que a aceitam ficam vulneráveis  
- Idealmente, cada cliente ou grupo de cliente deve ter sua própria chave  

---
# Boas práticas

- Gere pares de chaves diferentes para contextos distintos (trabalho, pessoal, automação)  
- Use um agente SSH para facilitar o uso de múltiplas chaves  
- Revogue imediatamente qualquer chave reutilizada que tenha sido exposta  
- Automatize a rotação periódica das chaves  

---
# <!--fit--> Dúvidas? 🤔