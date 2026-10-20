# Implementação de SPF, DKIM e DMARC contra Spoofing de Domínio

> **Caminho:** `cyber/email-dmarc/spf-dkim-dmarc.md`  
> **Objetivo:** Configurar os três protocolos de autenticação de e-mail no DNS público para impedir falsificação de remetente (*spoofing*) e melhorar a entregabilidade do domínio.

## 1. Pré-requisitos
- Acesso administrativo à zona DNS pública do domínio.
- Lista completa de servidores e serviços autorizados a enviar e-mail em nome do domínio (ex.: Microsoft 365, Google Workspace, plataforma de disparo transacional).

## 2. Passo a passo

### 2.1. Publicação do Registro SPF (Sender Policy Framework)
1. Crie um único registro `TXT` na raiz do domínio (`@`) listando apenas os serviços autorizados e finalizando com `-all` (*Hard Fail*):

```text
v=spf1 include:spf.protection.outlook.com -all
```

### 2.2. Ativação de Assinatura DKIM (DomainKeys Identified Mail)
1. No painel do provedor de e-mail (Microsoft 365 Defender ou Google Admin), gere as chaves DKIM de `2048 bits`.
2. Publique os registros `CNAME` ou `TXT` correspondentes aos seletores (ex.: `selector1._domainkey` e `selector2._domainkey`) no DNS público e ative a assinatura no painel.

### 2.3. Implementação Gradual do DMARC
1. Publique o registro `TXT` em `_dmarc.seudominio.com.br` iniciando em modo de monitoramento (`p=none`) e evoluindo para quarentena (`p=quarantine`) e rejeição (`p=reject`):

```text
v=DMARC1; p=reject; rua=mailto:dmarc-reports@seudominio.com.br; adkim=s; aspf=s; pct=100
```

## 3. Validação e Pontos de Atenção
- O protocolo SPF possui limite estrito de **10 consultas DNS** (*lookups*); evite encadear múltiplos `include:` desnecessários.
- Domínios estacionados que **não enviam e-mail** devem publicar `v=spf1 -all` e `v=DMARC1; p=reject;` para bloquear qualquer tentativa de uso indevido.
