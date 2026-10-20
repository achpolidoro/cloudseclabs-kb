# Teste de Envio SMTP Autenticado (Porta 587) via PowerShell

> **Caminho:** `modern-working/exchange-email/teste-envio-smtp.md`  
> **Objetivo:** Validar conectividade TLS, credenciais e permissão de envio `SMTP AUTH` na porta 587 para contas de serviço ou impressoras/sistemas legados.

---

## 1. Pré-requisitos

- Permissão `SmtpClientAuthenticationDisabled` definida como `$false` na caixa de serviço testada.
- Porta `587/TCP` liberada na saída de rede local.

---

## 2. Passo a passo

```powershell
# 1. Habilitar SMTP AUTH apenas na caixa de serviço (no Exchange Online PowerShell)
Set-CASMailbox -Identity "conta.servico@seudominio.com.br" -SmtpClientAuthenticationDisabled $false

# 2. Testar o disparo autenticado via STARTTLS na porta 587
$Credential = Get-Credential
Send-MailMessage -SmtpServer "smtp.office365.com" `
                 -Port 587 `
                 -UseSsl `
                 -From "conta.servico@seudominio.com.br" `
                 -To "destinatario@seudominio.com.br" `
                 -Subject "Teste de envio SMTP autenticado" `
                 -Body "Validação de envio SMTP AUTH via PowerShell (STARTTLS 587)." `
                 -Credential $Credential
```

---

## 3. Validação e Pontos de Atenção

1. Se o envio retornar erro `5.7.139 Authentication unsuccessful`, verifique se os **Security Defaults** estão ativos ou se uma política de Acesso Condicional está exigindo MFA interativo para essa conta.
2. Restrinja o login da conta de serviço exclusivamente aos IPs públicos conhecidos da organização via *Named Locations* no Acesso Condicional.
