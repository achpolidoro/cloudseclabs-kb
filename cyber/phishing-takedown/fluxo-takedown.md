# Fluxo de Resposta e Takedown de Domínios de Phishing

> **Caminho:** `cyber/phishing-takedown/fluxo-takedown.md`  
> **Objetivo:** Coletar evidências técnicas de páginas fraudulentas que imitam uma marca ou organização e acionar provedores de hospedagem, registrars e listas de bloqueio para remoção rápida (*takedown*).

## 1. Pré-requisitos
- Ferramentas de consulta `whois`, `dig` e `curl` (ou acesso ao `urlscan.io`).
- Canal de abuso dos principais registrars e provedores de nuvem.

## 2. Passo a passo

### 2.1. Coleta de Evidências Sem Alertar o Atacante
1. Submeta a URL suspeita em modo privado/unlisted no **urlscan.io** para capturar screenshot, árvore DOM, IP de hospedagem e certificado TLS emitido.
2. Consulte no terminal os dados de registro do domínio e o ASN responsável pelo endereço IP:

```bash
dig +short A dominio-falso.com
whois dominio-falso.com | grep -iE "Registrar|Abuse|Creation"
```

### 2.2. Notificação de Abuso e Inclusão em Blocklists Globais
1. Denuncie a URL diretamente às bases que alimentam os navegadores modernos:
   - **Google Safe Browsing:** `https://safebrowsing.google.com/safebrowsing/report_phish/`
   - **Microsoft SmartScreen:** `https://www.microsoft.com/wdsi/support/report-unsafe-site-guest`
   - **Cloudflare Abuse Form** (caso o domínio utilize nameservers ou proxy da Cloudflare): `https://abuse.cloudflare.com`
2. Envie notificação formal para o e-mail de *Registrar Abuse Contact* identificado no `whois`, anexando a captura de tela, cabeçalhos HTTP e comprovação de titularidade da marca legítima.

## 3. Validação e Pontos de Atenção
- Bloqueie preventivamente o domínio fraudulento no filtro DNS corporativo, firewall e gateway de e-mail interno enquanto o processo externo de takedown está em andamento.
- Verifique nos logs de autenticação (Entra ID / Google Workspace) se algum usuário interno clicou ou submeteu credenciais para o endereço malicioso.
