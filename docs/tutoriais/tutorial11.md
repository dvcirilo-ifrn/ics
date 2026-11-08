# Configuração de HTTPS com nginx e Let's Encrypt

Este tutorial apresenta como configurar um domínio gratuito e um certificado HTTPS do Let's Encrypt para um sistema Django servido pelo nginx em uma VM Debian (ex. Google Cloud).

!!! warning "Atenção!"
    Antes de seguir este tutorial, certifique-se de que você estudou o conteúdo da aula sobre HTTPS e certificados digitais. Este tutorial assume que o seu sistema já está funcionando com nginx e Gunicorn ([Tutorial 03](tutorial09.md)) e que a VM permite tráfego HTTP e HTTPS ([Tutorial 02](tutorial02.md)).

## Visão Geral

1. Criar um subdomínio gratuito no DuckDNS apontando para a VM;
2. Configurar o nginx e o Django para responder pelo domínio;
3. Obter o certificado com o *certbot*, que também configura o nginx;
4. Ativar as configurações de segurança do Django para HTTPS.

Os exemplos usam os seguintes nomes, que você deve substituir pelos seus:

| Nome | Significado |
|---|---|
| `<projeto>` | nome da pasta do seu projeto Django (o mesmo do Tutorial 03) |
| `<subdominio>` | nome escolhido no DuckDNS (ex. `meusite`, para `meusite.duckdns.org`) |
| `<token>` | *token* da sua conta no DuckDNS |
| `<ip-externo>` | IP externo da VM, exibido no console do Google Cloud |

## Parte 1: Domínio no DuckDNS

### Criando o subdomínio

1. Acesse [duckdns.org](https://www.duckdns.org) e entre com sua conta Google ou GitHub;
2. No campo *sub domain*, digite o nome desejado e clique em *add domain*;
3. No campo *current ip*, informe o IP externo da VM e clique em *update ip*;
4. Anote o *token* exibido no topo da página.

!!! danger "Importante!"
    O *token* permite alterar os seus domínios. Não o compartilhe nem o envie para o repositório.

### Atualizando o IP automaticamente

No Google Cloud, o IP externo da VM muda quando ela é desligada e ligada novamente. Para que o domínio acompanhe o IP, configure a VM para atualizar o DuckDNS periodicamente. Acesse a VM via SSH e abra o *crontab* do seu usuário:

```bash
crontab -e
```

Adicione as linhas abaixo ao final do arquivo, substituindo o subdomínio e o *token*:

```
@reboot sleep 30 && curl -s "https://www.duckdns.org/update?domains=<subdominio>&token=<token>&ip=" > /dev/null
*/5 * * * * curl -s "https://www.duckdns.org/update?domains=<subdominio>&token=<token>&ip=" > /dev/null
```

A primeira linha atualiza o IP 30 segundos após a VM ligar, e a segunda a cada 5 minutos. Com o `ip=` vazio, o DuckDNS usa o IP de quem fez a requisição, ou seja, o IP externo da VM.

Teste a atualização executando o comando manualmente. A resposta deve ser `OK`:

```bash
curl "https://www.duckdns.org/update?domains=<subdominio>&token=<token>&ip="
```

### Verificando o DNS

Verifique se o domínio aponta para o IP da VM:

```bash
getent hosts <subdominio>.duckdns.org
```

O comando deve exibir o IP externo da VM. Se não exibir, aguarde alguns minutos e tente novamente.

## Parte 2: Configurando o domínio no servidor

### Django

No arquivo `.env` do projeto, adicione o domínio ao `ALLOWED_HOSTS`:

```env
ALLOWED_HOSTS=<subdominio>.duckdns.org,<ip-externo>
```

Reinicie o Gunicorn para carregar o `.env`:

```bash
sudo systemctl restart <projeto>_gunicorn.service
```

### nginx

No arquivo `/etc/nginx/sites-available/<projeto>`, troque o `server_name`:

```nginx
server_name <subdominio>.duckdns.org;
```

O *certbot* usa o `server_name` para encontrar o site que deve receber o certificado.

Verifique a configuração e recarregue o nginx:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

Acesse `http://<subdominio>.duckdns.org` no navegador e verifique se o sistema abre.

## Parte 3: Certificado HTTPS

### Instalando o certbot

```bash
sudo apt install certbot python3-certbot-nginx
```

### Obtendo o certificado

```bash
sudo certbot --nginx -d <subdominio>.duckdns.org
```

Na primeira execução, o *certbot* pergunta:

- Um email para avisos importantes sobre a conta;
- Se você aceita os termos de uso (responda `Y`);
- Se deseja compartilhar o email com a EFF (opcional).

Em seguida, ele realiza o desafio HTTP-01, emite o certificado e altera a configuração do nginx. Ao final, deve aparecer a mensagem `Successfully deployed certificate`.

### O que o certbot alterou

Abra o arquivo `/etc/nginx/sites-available/<projeto>`. O *certbot* adicionou as linhas marcadas com `# managed by Certbot`:

```nginx
server {
    server_name <subdominio>.duckdns.org;
    client_max_body_size 10M;

    # ... locations do seu sistema ...

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/<subdominio>.duckdns.org/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/<subdominio>.duckdns.org/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}

server {
    if ($host = <subdominio>.duckdns.org) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

    listen 80;
    server_name <subdominio>.duckdns.org;
    return 404; # managed by Certbot
}
```

- O primeiro bloco passou a escutar na porta 443, com o certificado (`fullchain.pem`) e a chave privada (`privkey.pem`);
- O segundo bloco responde na porta 80 e redireciona (301) para o HTTPS.

Acesse `https://<subdominio>.duckdns.org` no navegador. O cadeado deve aparecer na barra de endereços. Teste também o redirecionamento:

```bash
curl -I http://<subdominio>.duckdns.org
```

A resposta deve ser `301 Moved Permanently`, com `Location: https://...`.

!!! warning "Acesso pelo IP"
    O certificado é válido apenas para o domínio. Acessar `https://<ip-externo>` exibe um alerta de segurança no navegador. Use sempre o domínio.

### Renovação automática

O pacote do *certbot* instala um *timer* do systemd que tenta renovar os certificados duas vezes por dia. Verifique se ele está ativo:

```bash
systemctl list-timers certbot.timer
```

Teste a renovação sem emitir um novo certificado:

```bash
sudo certbot renew --dry-run
```

!!! info "Limites do Let's Encrypt"
    O Let's Encrypt limita o número de tentativas e de certificados emitidos por domínio. Se algo der errado, corrija o problema antes de tentar novamente. Para testes, use `--dry-run` ou `--test-cert`, que usam o ambiente de testes do Let's Encrypt.

## Parte 4: Django com HTTPS

### Modificando o settings.py

Na máquina local, adicione ao final do `config/settings.py`:

```python
if os.getenv("USE_HTTPS") == "True":
    # Indica ao Django que a requisição chegou ao nginx via HTTPS
    SECURE_PROXY_SSL_HEADER = ("HTTP_X_FORWARDED_PROTO", "https")
    # Redireciona requisições HTTP para HTTPS
    SECURE_SSL_REDIRECT = True
    # Cookies de sessão e CSRF enviados apenas por HTTPS
    SESSION_COOKIE_SECURE = True
    CSRF_COOKIE_SECURE = True
    # HSTS: o navegador passa a usar apenas HTTPS por 1 hora
    SECURE_HSTS_SECONDS = 3600
```

Adicione a nova variável ao `.env.exemplo`:

```env
USE_HTTPS=False
```

Faça o *commit* e envie as alterações para o repositório.

### Atualizando o servidor

No servidor, atualize o projeto:

```bash
cd ~/<projeto>
git pull
```

Adicione ao `.env`:

```env
USE_HTTPS=True
```

Reinicie o Gunicorn:

```bash
sudo systemctl restart <projeto>_gunicorn.service
```

Acesse o sistema pelo domínio e faça login no `/admin` para verificar se tudo funciona.

### Verificando a configuração

```bash
source venv/bin/activate
python manage.py check --deploy
```

Os avisos de HTTPS do [Tutorial 03](tutorial09.md) devem ter desaparecido. Os avisos que restam são:

- `security.W005` e `security.W021`: sobre a inclusão de subdomínios e a lista de *preload* do HSTS. Não ative essas opções em um subdomínio do DuckDNS, pois elas afetam o domínio inteiro;
- `mail.E001`: o envio de email não foi configurado (veja a seção Email do [Tutorial 03](tutorial09.md#email-opcional)).

!!! warning "HSTS"
    Depois de alguns dias com tudo funcionando, aumente o `SECURE_HSTS_SECONDS` (ex. `31536000`, 1 ano). Não aumente antes: enquanto o tempo não expirar, o navegador recusa acessar o site sem HTTPS, mesmo que você desative o certificado.

## Solução de Problemas

### Onde ver os erros

```bash
# Erros do certbot
sudo tail -n 50 /var/log/letsencrypt/letsencrypt.log

# Erros do nginx
sudo tail /var/log/nginx/error.log

# Erros do Django/Gunicorn
sudo journalctl -u <projeto>_gunicorn.service -f
```

### Erros comuns

| Erro | Causa provável |
|---|---|
| *certbot*: não encontra o site (*server block*) | O `server_name` no nginx não é igual ao domínio informado no `-d` |
| *certbot*: *Timeout* ou *Connection refused* na validação | A porta 80 está bloqueada no *firewall* da VM ou o domínio não aponta para a VM |
| *certbot*: *DNS problem: NXDOMAIN* | O domínio foi digitado errado ou ainda não existe no DuckDNS |
| *certbot*: *too many failed authorizations* | Limite de tentativas do Let's Encrypt. Aguarde 1 hora e corrija o problema antes de tentar de novo |
| 400 Bad Request | O domínio não está no `ALLOWED_HOSTS` do `.env` |
| Site não abre depois de religar a VM | O IP mudou e o DuckDNS ainda não foi atualizado. Aguarde 5 minutos ou execute o `curl` de atualização manualmente |
| HTTPS não abre, mas HTTP funciona | A porta 443 não está liberada (**Permitir tráfego HTTPS** no Google Cloud) |
| Alerta de certificado inválido | O acesso foi feito pelo IP, e não pelo domínio |
| `curl --unix-socket` do Tutorial 03 retorna 301 | Esperado com `USE_HTTPS=True`: o Django redireciona para HTTPS |
