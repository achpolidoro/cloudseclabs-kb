# Endereçamento Dinâmico com "+" (Plus Addressing) no Exchange

> **Caminho:** `modern-working/exchange-email/plus-addressing-aliases.md`  
> **Objetivo:** Verificar e ativar o suporte a *Plus Addressing* (`usuario+tag@seudominio.com.br`) no Exchange Online para triagem e rastreamento de mensagens sem criar aliases manuais.

---

## 1. Pré-requisitos

- Módulo `ExchangeOnlineManagement` instalado no PowerShell.
- Permissão de **Exchange Administrator**.

---

## 2. Passo a passo

```powershell
# 1. Conectar ao Exchange Online
Connect-ExchangeOnline

# 2. Verificar se o Plus Addressing está ativo no tenant (False = ativo)
Get-OrganizationConfig | Select-Object DisablePlusAddressInRecipients

# 3. Habilitar o suporte a Plus Addressing caso esteja desativado
Set-OrganizationConfig -DisablePlusAddressInRecipients $false
```

---

## 3. Validação e Pontos de Atenção

1. Envie um e-mail externo de teste para `sua.conta+lab@seudominio.com.br`.
2. A mensagem deve chegar normalmente na caixa principal de `sua.conta@seudominio.com.br`, preservando o cabeçalho `To:` com a tag `+lab` para criação de regras de filtro.
