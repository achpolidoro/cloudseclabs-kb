# Execução de Assessment M365 via PowerShell e Graph

> **Caminho:** `modern-working/assessment-conformidade/execucao-assessment-m365.md`  
> **Objetivo:** Executar a coleta automatizada de configurações de segurança no Microsoft 365 e tratar armadilhas comuns de interpretação nos relatórios gerados.

---

## 1. Pré-requisitos

- App Registration configurado com as permissões de leitura e consentimento administrativo concedido.
- Módulos PowerShell instalados na máquina de coleta.

---

## 2. Passo a passo

### 2.1 Validação Prévia e Execução da Coleta

```powershell
# 1. Validar dependências e listar coletores disponíveis sem autenticar
./Invoke-M365Assessment.ps1 -ValidateOnly
./Invoke-M365Assessment.ps1 -DryRun

# 2. Executar a coleta informando TenantId e ClientId
$secret = Read-Host "Informe o Client Secret" -AsSecureString

./Invoke-M365Assessment.ps1 `
    -AuthenticationMode AppOnly `
    -TenantId "<TENANT_ID>" `
    -ClientId "<CLIENT_ID>" `
    -ClientSecret $secret `
    -StartDate (Get-Date).AddDays(-30)
```

### 2.2 Verificação do Resumo de Execução

```powershell
# Conferir total de coletores com sucesso vs. falha
Get-Content ./output/summary.json | ConvertFrom-Json | Select-Object totals

# Detalhar eventuais falhas
(Get-Content ./output/summary.json -Raw | ConvertFrom-Json).results |
    Where-Object status -eq 'failed' |
    Select-Object name, error | Format-List
```

### 2.3 Coleta Complementar no Exchange Online

Caso a coleta seja feita apenas com *Client Secret* (sem certificado), execute manualmente com uma conta `Global Reader` / `View-Only Recipients`:

```powershell
Connect-ExchangeOnline -UserPrincipalName "admin@seudominio.com.br"

Get-EXOMailbox -ResultSize Unlimited `
    -Properties ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward,AuditEnabled,LitigationHoldEnabled |
    Where-Object { $_.ForwardingAddress -or $_.ForwardingSmtpAddress } |
    Export-Csv .\output\exchange-forwarding.csv -NoTypeInformation -Encoding UTF8

Get-OrganizationConfig | Select-Object AuditDisabled
```

---

## 3. Validação e Pontos de Atenção

1. **Cache de Token no PowerShell:** Se uma permissão for adicionada no portal após uma falha `403 Forbidden`, feche a janela do PowerShell, aguarde 5 minutos e abra uma nova sessão antes de reexecutar.
2. **Falso negativo de DKIM:** Scripts que consultam apenas `selector1` e `selector2` reportam DKIM ausente caso o domínio utilize seletores personalizados. Confirme diretamente no portal Defender.
3. **Dependência de Licenciamento:** Coletores de *Sign-in Logs* exigem **Entra ID P1** e coletores de *Identity Protection / PIM* exigem **Entra ID P2**.
