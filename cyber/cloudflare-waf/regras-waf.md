# Regras Essenciais de WAF e Rate Limiting na Cloudflare

> **Caminho:** `cyber/cloudflare-waf/regras-waf.md`  
> **Objetivo:** Configurar regras customizadas de Web Application Firewall (WAF) na Cloudflare para bloquear varreduras automatizadas, proteger rotas de login e restringir países ou ASNs maliciosos.

## 1. Pré-requisitos
- Domínio ativo na Cloudflare com registros DNS em modo **Proxied** (nuvem laranja ativa).
- Acesso ao painel **Security** > **WAF** da zona.

## 2. Passo a passo

### 2.1. Criação de Custom Rules para Proteção de Caminhos Sensíveis
1. No painel da Cloudflare, acesse **Security** > **WAF** > **Custom rules** > **Create rule**.
2. Para bloquear acesso a arquivos de configuração, diretórios `.git` e rotas administrativas legadas, clique em **Edit expression** e utilize:

```text
(http.request.uri.path contains "/.git") or
(http.request.uri.path contains "/.env") or
(http.request.uri.path contains "/wp-config.php") or
(http.request.uri.path contains "/xmlrpc.php")
```

3. Em **Choose action**, selecione **Block** e clique em **Deploy**.

### 2.2. Desafio Gerenciado (Managed Challenge) para Rotas de Autenticação
1. Crie uma segunda regra para exigir verificação em áreas de login quando a requisição vier de fora do Brasil ou com *Threat Score* elevado:

```text
(http.request.uri.path contains "/login" or http.request.uri.path contains "/admin") and
(ip.geoip.country ne "BR")
```

2. Selecione a ação **Managed Challenge** e salve.

## 3. Validação e Pontos de Atenção
- Acesse **Security** > **Events** após 24 horas para auditar os bloqueios e garantir que integrações legítimas (webhooks ou APIs externas) não foram impactadas.
- Ative **Bot Fight Mode** em **Security** > **Bots** para mitigar scrapers simples sem consumir cota de Custom Rules.
