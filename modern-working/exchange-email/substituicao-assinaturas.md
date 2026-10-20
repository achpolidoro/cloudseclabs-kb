# Substituição de Assinaturas de E-mail em Massa via PowerShell

> **Caminho:** `modern-working/exchange-email/substituicao-assinaturas.md`  
> **Objetivo:** Aplicar assinaturas HTML padronizadas em todas as caixas de correio do Exchange Online via PowerShell 7, tratando o comportamento de *roaming signatures*.

---

## 1. Pré-requisitos

- **PowerShell 7** com módulo `ExchangeOnlineManagement`.
- Permissão de **Exchange Administrator**.
- Arquivos `.html` de assinatura salvos em `C:\temp\assinaturas\` com o nome exato do UPN de cada usuário (ex.: `usuario@seudominio.com.br.html`).

---

## 2. Passo a passo

### 2.1 Conectar e Adiar Roaming Signatures no Tenant

Quando o recurso *Roaming Signatures* está ativo, o Outlook ignora o atributo gravado pelo cmdlet `Set-MailboxMessageConfiguration`. Para que o script tenha efeito:

```powershell
Install-Module ExchangeOnlineManagement -Scope CurrentUser -Force
Connect-ExchangeOnline

# Adiar Roaming Signatures no nível da organização
Set-OrganizationConfig -PostponeRoamingSignaturesUntilLater $true
Get-OrganizationConfig | Format-List PostponeRoamingSignaturesUntilLater
```

### 2.2 Testar em uma Única Caixa e Aplicar em Lote

```powershell
# 1. Teste individual antes do lote
$teste = "usuario.teste@seudominio.com.br"
Set-MailboxMessageConfiguration -Identity $teste `
    -SignatureHtml (Get-Content "C:\temp\assinaturas\$teste.html" -Raw) `
    -AutoAddSignature $true `
    -AutoAddSignatureOnReply $true

# 2. Aplicação em massa com geração de log CSV
$pasta = "C:\temp\assinaturas\"
$log = @()

foreach ($item in Get-ChildItem -Path $pasta -Filter *.html) {
    $usuario = $item.BaseName
    try {
        Set-MailboxMessageConfiguration -Identity $usuario `
            -SignatureHtml (Get-Content $item.FullName -Raw) `
            -AutoAddSignature $true `
            -AutoAddSignatureOnReply $true `
            -ErrorAction Stop
        $status = "OK"; $erro = ""
    }
    catch {
        $status = "ERRO"; $erro = $_.Exception.Message
    }

    Write-Host "$status - $usuario"
    $log += [pscustomobject]@{ Usuario = $usuario; Status = $status; Erro = $erro }
    Start-Sleep -Milliseconds 300
}

$log | Export-Csv .\log-assinaturas.csv -NoTypeInformation -Encoding UTF8
Disconnect-ExchangeOnline -Confirm:$false
```

---

## 3. Validação e Pontos de Atenção

1. Aguarde de 15 a 60 minutos após alterar `PostponeRoamingSignaturesUntilLater` e valide no **Outlook Web** em janela anônima clicando em **Novo e-mail**.
2. Imagens da assinatura devem estar hospedadas em URL HTTPS pública (evite imagens em Base64, que são bloqueadas por clientes de e-mail).
