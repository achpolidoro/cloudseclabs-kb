# Teste de Envio SMTP Autenticado (Porta 587 / STARTTLS) via PowerShell

> **Objetivo:** Validar rapidamente credenciais e permissão de envio (`SMTP AUTH`) de uma caixa ou conta de serviço via linha de comando.

## 1. Comando de Teste no PowerShell

```powershell
$Credential = Get-Credential
Send-MailMessage -SmtpServer "smtp.office365.com" `
                 -Port 587 `
                 -UseSsl `
                 -From "conta.servico@seudominio.com.br" `
                 -To "destinatario@seudominio.com.br" `
                 -Subject "Teste de envio SMTP autenticado" `
                 -Body "Mensagem de validação enviada via PowerShell (STARTTLS 587)." `
                 -Credential $Credential
```

## 2. Observações Técnicas
- Caso o `SmtpClientAuthenticationDisabled` esteja como `$true` na organização, habilite apenas na caixa de serviço específica:
```powershell
Set-CASMailbox -Identity "conta.servico@seudominio.com.br" -SmtpClientAuthenticationDisabled $false
```
- Contas usadas para `SMTP AUTH` básico devem estar excluídas de políticas de MFA interativo ou restritas por IP Confiável (*Named Locations*) no Acesso Condicional.
