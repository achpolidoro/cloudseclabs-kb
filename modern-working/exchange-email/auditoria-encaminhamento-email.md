# Auditoria de Encaminhamento de E-mail e Regras de Caixa de Entrada

#### Objetivo

Fornecer um passo a passo seguro e direto para **instalar o módulo ExchangeOnline**, **conectar-se** e **listar na tela** apenas as mailboxes que possuem encaminhamento configurado (encaminhamento na mailbox ou regras de inbox).

### Pré requisitos

- **Conta com permissões de leitura** em Exchange Online (recomendado: View‑Only Recipients ou role customizada com Get‑Mailbox e leitura de regras).
    
- **PowerShell 7+** ou Windows PowerShell 5.1.
    
- Conexão de rede com acesso ao Microsoft 365.
    
- Executar em máquina corporativa segura.
    

### Passos operacionais

#### 1 Instalar e importar o módulo

```powershell
# Executar como administrador se necessário
Install-Module -Name ExchangeOnlineManagement -Scope CurrentUser -Force

# Importar módulo
Import-Module ExchangeOnlineManagement
```

#### 2 Conectar ao Exchange Online

```powershell
# Conexão interativa (suporta MFA)
Connect-ExchangeOnline -UserPrincipalName seu.email@dominio.com
```

**Observação**: para automação sem interação use autenticação por certificado e App Registration (requer configuração prévia no Azure AD).

#### 3 Rodar script simples para listar mailboxes com encaminhamento

```powershell
# Executar após Connect-ExchangeOnline

Get-Mailbox -ResultSize Unlimited | ForEach-Object {
    $mb = $_
    $upn = $mb.UserPrincipalName
    $display = $mb.DisplayName

    # Encaminhamento configurado na mailbox
    $fwSmtp = if ($mb.ForwardingSmtpAddress) { $mb.ForwardingSmtpAddress.ToString() } else { $null }
    $fwAddr = if ($mb.ForwardingAddress) { $mb.ForwardingAddress.ToString() } else { $null }
    $deliverAndForward = $mb.DeliverToMailboxAndForward

    # Regras de inbox que fazem forward/redirect
    $inboxTargets = @()
    try {
        $rules = Get-InboxRule -Mailbox $upn -ErrorAction Stop
        foreach ($r in $rules) {
            if ($r.Enabled) {
                if ($r.ForwardTo -and $r.ForwardTo.Count -gt 0) {
                    $inboxTargets += ($r.ForwardTo | ForEach-Object { if ($_.SmtpAddress) { $_.SmtpAddress } elseif ($_.Address) { $_.Address } else { $_.ToString() } })
                }
                if ($r.RedirectTo -and $r.RedirectTo.Count -gt 0) {
                    $inboxTargets += ($r.RedirectTo | ForEach-Object { if ($_.SmtpAddress) { $_.SmtpAddress } elseif ($_.Address) { $_.Address } else { $_.ToString() } })
                }
            }
        }
    } catch {
        # sem permissão ou sem regras -> ignora
    }

    $inboxTargets = $inboxTargets | Where-Object { $_ } | Select-Object -Unique

    if ($fwSmtp -or $fwAddr -or $inboxTargets.Count -gt 0) {
        Write-Host "----------------------------------------"
        Write-Host "DisplayName: $display"
        Write-Host "UserPrincipalName: $upn"
        Write-Host "DeliverToMailboxAndForward: $deliverAndForward"
        if ($fwSmtp) { Write-Host "Mailbox Forwarding SMTP: $fwSmtp" }
        if ($fwAddr) { Write-Host "Mailbox Forwarding Recipient: $fwAddr" }
        if ($inboxTargets.Count -gt 0) {
            Write-Host "Inbox Rule Forward/Redirect Targets:"
            $inboxTargets | ForEach-Object { Write-Host "  - $_" }
        }
    }
}
```

### Saferunning e recomendações

- **O script é somente leitura**; não altera configurações.
    
- **Trate a saída como sensível**: contém endereços internos e possivelmente externos. Armazene/exporte com segurança.
    
- **Teste primeiro** em 1–2 mailboxes antes de rodar em todo o tenant.
    
- Se preferir limitar o escopo, substitua `Get-Mailbox -ResultSize Unlimited` por um filtro, por exemplo:
    

```powershell
Get-Mailbox -Filter "RecipientTypeDetails -eq 'UserMailbox'" -ResultSize 200
```

- Se houver falhas de permissão em `Get-InboxRule`, execute com uma conta que tenha permissão ou rode apenas a verificação de propriedades da mailbox.
    

### Encerramento da sessão

```powershell
# Desconectar sem prompt de confirmação
Disconnect-ExchangeOnline -Confirm:$false
```
