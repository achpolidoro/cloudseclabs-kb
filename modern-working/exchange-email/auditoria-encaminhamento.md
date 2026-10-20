# Auditoria de Encaminhamento de E-mail e Regras de Inbox

> **Caminho:** `modern-working/exchange-email/auditoria-encaminhamento.md`  
> **Objetivo:** Identificar todas as caixas de correio do Exchange Online que possuem redirecionamento configurado nas propriedades da conta (`ForwardingSmtpAddress`) ou em regras de caixa de entrada (`Inbox Rules`).

---

## 1. Pré-requisitos

- **PowerShell 7+** com módulo `ExchangeOnlineManagement`.
- Conta com permissão de leitura (`View-Only Recipients` ou `Exchange Administrator`).

---

## 2. Passo a passo

```powershell
# 1. Conectar ao Exchange Online
Import-Module ExchangeOnlineManagement
Connect-ExchangeOnline -UserPrincipalName "admin@seudominio.com.br"

# 2. Varrer todas as caixas e regras de entrada ativas
Get-Mailbox -ResultSize Unlimited | ForEach-Object {
    $mb = $_
    $upn = $mb.UserPrincipalName
    $display = $mb.DisplayName

    $fwSmtp = if ($mb.ForwardingSmtpAddress) { $mb.ForwardingSmtpAddress.ToString() } else { $null }
    $fwAddr = if ($mb.ForwardingAddress) { $mb.ForwardingAddress.ToString() } else { $null }
    $deliverAndForward = $mb.DeliverToMailboxAndForward

    $inboxTargets = @()
    try {
        $rules = Get-InboxRule -Mailbox $upn -ErrorAction Stop
        foreach ($r in $rules) {
            if ($r.Enabled) {
                if ($r.ForwardTo) {
                    $inboxTargets += ($r.ForwardTo | ForEach-Object { $_.ToString() })
                }
                if ($r.RedirectTo) {
                    $inboxTargets += ($r.RedirectTo | ForEach-Object { $_.ToString() })
                }
            }
        }
    } catch {}

    $inboxTargets = $inboxTargets | Where-Object { $_ } | Select-Object -Unique

    if ($fwSmtp -or $fwAddr -or $inboxTargets.Count -gt 0) {
        Write-Host "----------------------------------------"
        Write-Host "Caixa: $display ($upn)"
        Write-Host "Mantém cópia local: $deliverAndForward"
        if ($fwSmtp) { Write-Host "ForwardingSmtpAddress: $fwSmtp" }
        if ($fwAddr) { Write-Host "ForwardingAddress: $fwAddr" }
        if ($inboxTargets.Count -gt 0) {
            Write-Host "Regras de Inbox (Forward/Redirect):"
            $inboxTargets | ForEach-Object { Write-Host "  -> $_" }
        }
    }
}

# 3. Encerrar sessão
Disconnect-ExchangeOnline -Confirm:$false
```

---

## 3. Validação e Pontos de Atenção

1. Revise qualquer destino externo desconhecido: regras de caixa de entrada que encaminham mensagens para fora do domínio são um indicador clássico de comprometimento de conta (BEC).
2. Combine esta auditoria com o bloqueio de encaminhamento externo automático na política *Outbound Spam Filter* do Defender.
