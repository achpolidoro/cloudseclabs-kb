# Gerenciamento de Aliases e SMTP Primário via PowerShell

> **Caminho:** `modern-working/identidade-sso/alterar-aliases-smtp.md`  
> **Objetivo:** Consultar, adicionar ou remover apelidos de e-mail (*aliases*) e alterar o endereço SMTP principal de uma caixa no Exchange Online / Entra ID.

---

## 1. Pré-requisitos

- Módulo `ExchangeOnlineManagement` instalado no PowerShell.
- Permissão de **Exchange Administrator** ou **User Administrator**.

---

## 2. Passo a passo

```powershell
# 1. Instalar módulo e conectar ao Exchange Online
Install-Module ExchangeOnlineManagement -Scope CurrentUser
Connect-ExchangeOnline

# 2. Consultar os endereços atuais da caixa
$upn = "usuario@seudominio.com.br"
Get-Mailbox -Identity $upn | Select-Object DisplayName, PrimarySmtpAddress, EmailAddresses

# 3. Adicionar um novo alias secundário (smtp minúsculo)
Set-Mailbox -Identity $upn -EmailAddresses @{Add="alias.novo@seudominio.com.br"}

# 4. Remover um alias existente
Set-Mailbox -Identity $upn -EmailAddresses @{Remove="alias.antigo@seudominio.com.br"}

# 5. (Opcional) Alterar o endereço SMTP principal de envio
Set-Mailbox -Identity $upn -PrimarySmtpAddress "principal.novo@seudominio.com.br"
```

---

## 3. Validação e Pontos de Atenção

1. Execute novamente `Get-Mailbox -Identity $upn | Select-Object PrimarySmtpAddress, EmailAddresses` para confirmar que o endereço principal consta com prefixo `SMTP:` (maiúsculo) e os secundários com `smtp:` (minúsculo).
2. Em ambientes sincronizados com Active Directory local (Entra Connect), a alteração do atributo `proxyAddresses` deve ser feita no AD local.
