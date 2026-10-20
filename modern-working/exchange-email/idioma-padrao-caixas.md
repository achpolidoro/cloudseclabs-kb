# Definir Idioma Padrão (pt-BR) nas Caixas do Exchange Online

> **Caminho:** `modern-working/exchange-email/idioma-padrao-caixas.md`  
> **Objetivo:** Configurar o idioma regional (`pt-BR`) e traduzir os nomes das pastas padrão (*Inbox* para *Caixa de Entrada*) individualmente ou em massa via PowerShell.

---

## 1. Pré-requisitos

- Módulo `ExchangeOnlineManagement` instalado no PowerShell.
- Permissão de **Exchange Administrator**.

---

## 2. Passo a passo

### 2.1 Aplicar em uma Única Caixa de Correio

```powershell
Connect-ExchangeOnline

Set-MailboxRegionalConfiguration -Identity "usuario@seudominio.com.br" `
    -Language pt-BR `
    -LocalizeDefaultFolderName:$true

Get-MailboxRegionalConfiguration -Identity "usuario@seudominio.com.br" |
    Select-Object Identity, Language
```

### 2.2 Aplicar em Massa para Todas as Caixas de Usuários

```powershell
$caixas = Get-Mailbox -ResultSize Unlimited -RecipientTypeDetails UserMailbox

foreach ($caixa in $caixas) {
    Set-MailboxRegionalConfiguration -Identity $caixa.UserPrincipalName `
        -Language pt-BR `
        -LocalizeDefaultFolderName:$true
}
```

---

## 3. Validação e Pontos de Atenção

1. Para listar o idioma configurado em todas as caixas do tenant:
```powershell
Get-Mailbox -ResultSize Unlimited |
    Get-MailboxRegionalConfiguration |
    Select-Object Identity, Language
```
2. O parâmetro `-LocalizeDefaultFolderName:$true` renomeia pastas nativas em inglês (*Sent Items*, *Deleted Items*) para o idioma definido.
