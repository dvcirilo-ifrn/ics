# Deploy automático com GitHub Actions

Este tutorial apresenta como configurar um *workflow* do GitHub Actions que testa o projeto Django e faz o deploy automático em uma VM do Google Cloud com Docker Compose a cada *push* na *branch* `main`.

!!! warning "Atenção!"
    Antes de seguir este tutorial, certifique-se de que você estudou o conteúdo da aula sobre implantação automática. Este tutorial assume que o seu sistema já está funcionando na VM com Docker Compose ([Tutorial 04](tutorial10.md)), a partir de um repositório no GitHub, e que o seu usuário na VM pertence ao grupo `docker`.

## Visão Geral

A cada *push* na *branch* `main`, o *workflow*:

1. **Job `test`**: instala as dependências em um *runner* do GitHub e executa o `check` e os testes do Django;
2. **Job `deploy`**: se os testes passarem, acessa a VM via SSH, atualiza o código com `git pull` e reconstrói os containers com `docker compose up -d --build`.

Os exemplos usam os seguintes nomes, que você deve substituir pelos seus:

| Nome | Significado |
|---|---|
| `<projeto>` | nome da pasta do projeto na VM (a mesma do Tutorial 04) |
| `<user>` | seu usuário na VM (descubra com o comando `whoami`) |
| `<servidor>` | IP externo da VM ou domínio do DuckDNS ([Tutorial 05](tutorial11.md)) |

!!! info "IP externo"
    O IP externo da VM muda quando ela é desligada e ligada novamente. Se você usar o IP, terá que atualizar o *secret* `SSH_HOST` a cada mudança. Com um domínio do DuckDNS atualizado automaticamente, isso não é necessário.

## Parte 1: Chave SSH para o deploy

O GitHub Actions precisa de uma chave SSH para acessar a VM. Crie uma chave exclusiva para isso, sem senha, já que o *workflow* não tem como digitá-la.

Acesse a VM via SSH e gere a chave:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/github_actions -N "" -C "github-actions"
```

Autorize a chave pública a acessar o seu usuário:

```bash
cat ~/.ssh/github_actions.pub >> ~/.ssh/authorized_keys
```

Exiba a chave privada e copie todo o conteúdo, incluindo as linhas `-----BEGIN OPENSSH PRIVATE KEY-----` e `-----END OPENSSH PRIVATE KEY-----`:

```bash
cat ~/.ssh/github_actions
```

Exiba também a chave pública do servidor e copie o conteúdo:

```bash
cat /etc/ssh/ssh_host_ed25519_key.pub
```

Ela permite que o *workflow* verifique que está se conectando à sua VM, e não a um servidor falso.

Depois de cadastrar os *secrets* (Parte 2), apague a chave privada da VM. Ela só é necessária no GitHub:

```bash
rm ~/.ssh/github_actions
```

!!! danger "Importante!"
    A chave privada dá acesso à sua VM. Nunca a coloque no repositório ou em qualquer arquivo do projeto.

## Parte 2: Secrets no GitHub

No repositório do GitHub, acesse **Settings** > **Secrets and variables** > **Actions** e clique em **New repository secret** para cadastrar cada um dos *secrets*:

| Nome | Valor |
|---|---|
| `SSH_HOST` | `<servidor>`: IP externo da VM ou domínio |
| `SSH_USER` | `<user>`: seu usuário na VM |
| `SSH_PRIVATE_KEY` | conteúdo completo da chave privada (`~/.ssh/github_actions`) |
| `SSH_HOST_KEY` | conteúdo da chave pública do servidor (`/etc/ssh/ssh_host_ed25519_key.pub`) |

Os *secrets* não podem ser lidos depois de cadastrados, apenas substituídos. Nos *logs* do *workflow*, eles aparecem como `***`.

## Parte 3: Workflow

Na máquina local, crie o arquivo `.github/workflows/deploy.yml` na raiz do projeto:

```yaml
name: CI/CD

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Baixa o código
        uses: actions/checkout@v7

      - name: Instala o Python
        uses: actions/setup-python@v7
        with:
          python-version: "3.14"

      - name: Instala as dependências
        run: pip install -r requirements.txt

      - name: Verifica o projeto
        run: python manage.py check

      - name: Executa os testes
        run: python manage.py test

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Configura o SSH
        env:
          SSH_HOST: ${{ secrets.SSH_HOST }}
          SSH_HOST_KEY: ${{ secrets.SSH_HOST_KEY }}
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
        run: |
          mkdir -p ~/.ssh
          echo "$SSH_PRIVATE_KEY" > ~/.ssh/id_ed25519
          chmod 600 ~/.ssh/id_ed25519
          echo "$SSH_HOST $SSH_HOST_KEY" > ~/.ssh/known_hosts

      - name: Atualiza o servidor
        env:
          SSH_HOST: ${{ secrets.SSH_HOST }}
          SSH_USER: ${{ secrets.SSH_USER }}
        run: |
          ssh "$SSH_USER@$SSH_HOST" "cd ~/<projeto> && git pull && docker compose up -d --build"
```

Substitua `<projeto>` pelo nome da pasta do projeto na VM.

### Entendendo o workflow

- `on`: o *workflow* roda a cada *push* na `main` e também manualmente (`workflow_dispatch`);
- `test`: roda em uma máquina Ubuntu nova, com a mesma versão do Python do `Dockerfile`:
    - Instala o `requirements.txt`. As dependências de produção (`requirements-prod.txt`) não são necessárias para os testes;
    - Sem o `.env`, o Django usa as configurações padrão, com SQLite;
- `deploy`: com `needs: test`, só roda se o *job* `test` terminar sem erros:
    - O primeiro passo grava a chave privada e a chave do servidor no *runner*;
    - O segundo acessa a VM e executa os mesmos comandos de "Atualizando o Projeto" do Tutorial 04;
- Os *secrets* são passados por variáveis de ambiente (`env`), e não diretamente nos comandos, para evitar problemas com caracteres especiais.

!!! warning "Branch"
    Se a *branch* principal do seu repositório se chama `master`, troque `main` por `master` no `on.push.branches`.

Faça o *commit* do arquivo e envie para o repositório:

```bash
git add .github/workflows/deploy.yml
git commit -m "Adiciona workflow de CI/CD"
git push
```

## Parte 4: Executando

O *push* dispara o *workflow*. No repositório do GitHub, acesse a aba **Actions** para acompanhar a execução:

- Cada execução mostra os *jobs* `test` e `deploy`;
- Clique em um *job* para ver a saída de cada passo;
- Um ✓ verde indica sucesso; um ✗ vermelho indica falha.

Quando o *deploy* terminar, acesse o sistema no navegador e verifique se a alteração foi publicada.

Para executar o *workflow* sem um novo *push*, acesse **Actions** > **CI/CD** > **Run workflow**.

!!! info "Indisponibilidade"
    Durante o `docker compose up -d --build`, o container `web` é recriado e o sistema fica indisponível por alguns segundos.

### Badge de status (opcional)

Para exibir o status do *workflow* no `README.md` do repositório, adicione a linha (substituindo o usuário e o repositório do GitHub):

```markdown
![CI/CD](https://github.com/<usuario-github>/<repositorio>/actions/workflows/deploy.yml/badge.svg)
```

## Desfazendo um deploy

Se uma alteração com problema for publicada, desfaça o *commit* na máquina local e envie para o repositório. O *workflow* publica a versão anterior:

```bash
git revert HEAD
git push
```

## Solução de Problemas

| Erro | Causa provável |
|---|---|
| *Job* `test` falhou | Erro no projeto ou nos testes. O `deploy` não é executado, e a versão anterior continua no ar |
| `Permission denied (publickey)` | A chave pública não está no `authorized_keys`, o `SSH_USER` está errado ou a chave privada foi copiada incompleta |
| `Host key verification failed` | O `SSH_HOST_KEY` está errado ou o `SSH_HOST` não corresponde ao endereço usado |
| `Connection timed out` | A VM está desligada ou o IP mudou e o `SSH_HOST` não foi atualizado |
| `permission denied ... docker.sock` | O usuário não pertence ao grupo `docker` na VM |
| `git pull` falhou | Há alterações feitas diretamente nos arquivos da VM, ou o repositório é privado e a VM não tem acesso a ele |
| *Workflow* não foi disparado | O arquivo não está em `.github/workflows/` ou o *push* foi feito em outra *branch* |
