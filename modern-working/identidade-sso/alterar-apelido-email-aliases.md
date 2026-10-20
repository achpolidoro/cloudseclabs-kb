# Gerenciamento de Aliases e Endereço SMTP Principal via PowerShell

Procedimento rápido para listar, adicionar ou remover apelidos de e-mail (*aliases*) e alterar o endereço SMTP primário no Exchange Online / Entra ID.


```powershell
# 1) Conectar no Exchange Online
Install-Module ExchangeOnlineManagement -Scope CurrentUser
Connect-ExchangeOnline

# 2) Ver os endereços atuais
$upn = "usuario@seu-dominio.com"
Get-Mailbox -Identity $upn | Select-Object DisplayName,PrimarySmtpAddress,EmailAddresses

# 3) Adicionar um alias (não-primário)
Set-Mailbox -Identity $upn -EmailAddresses @{Add="alias.novo@seu-dominio.com"}

# 4) Remover um alias
Set-Mailbox -Identity $upn -EmailAddresses @{Remove="alias.antigo@seu-dominio.com"}

# 5) (Opcional) Trocar o endereço principal (maiúsculo SMTP:)
#    ATENÇÃO: isso muda o remetente padrão do usuário
Set-Mailbox -Identity $upn -PrimarySmtpAddress "principal.novo@seu-dominio.com"

# 6) Conferir novamente
Get-Mailbox -Identity $upn | Select-Object DisplayName,PrimarySmtpAddress,EmailAddresses

```
