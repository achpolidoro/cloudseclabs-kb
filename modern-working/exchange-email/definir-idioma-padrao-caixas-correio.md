# Definir Idioma Padrão de Caixas de Correio no Exchange Online

> **Objetivo:** Padronizar o idioma e formato regional (`pt-BR`) de caixas de correio individualmente ou em lote via PowerShell.

## 1. Alterar idioma de um único usuário

```powershell
Connect-ExchangeOnline

# Definir idioma Português (Brasil)
Set-MailboxRegionalConfiguration -Identity "usuario@seudominio.com.br" -Language pt-BR -LocalizeDefaultFolderName:$true

# Validar configuração aplicada
Get-MailboxRegionalConfiguration -Identity "usuario@seudominio.com.br" | Select-Object Identity, Language
```

## 2. Alterar idioma para todas as caixas do ambiente

> **Atenção:** Execute em lote apenas quando houver necessidade de padronizar todas as caixas do diretório.

```powershell
$users = Get-Mailbox -ResultSize Unlimited -RecipientTypeDetails UserMailbox
foreach ($user in $users) {
    Set-MailboxRegionalConfiguration -Identity $user.UserPrincipalName -Language pt-BR -LocalizeDefaultFolderName:$true
}
```

## 3. Auditar idioma atual de todas as caixas

```powershell
Get-Mailbox -ResultSize Unlimited | Get-MailboxRegionalConfiguration |
    Select-Object Identity, Language
```
