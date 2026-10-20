# Revogação Global de Sessões e Refresh Tokens no Entra ID (M365)

> **Objetivo:** Forçar a invalidação imediata dos *Refresh Tokens* de todos os usuários do diretório via **PowerShell 7** e **Microsoft Graph**, preservando as contas administrativas de emergência (*break-glass*).

## 1. Pré-requisitos

- **PowerShell 7+**
- Módulo `Microsoft.Graph` instalado (`Install-Module Microsoft.Graph -Scope CurrentUser`)
- Permissão `User.ReadWrite.All` e `Directory.ReadWrite.All`

## 2. Script de Execução

```powershell
# 1. Conectar ao Microsoft Graph (substitua pelo domínio ou TenantId do ambiente)
Connect-MgGraph -TenantId "seudominio.onmicrosoft.com" `
                -Scopes "User.ReadWrite.All", "Directory.ReadWrite.All"

# 2. Lista de exclusão (Admin Global / Contas Break-Glass que NÃO devem ser desconectadas)
$contasIgnoradas = @(
    "admin.emergencia@seudominio.onmicrosoft.com"
)

# 3. Revogação em lote
$todosUsuarios = Get-MgUser -All
Write-Host "Iniciando revogação para $($todosUsuarios.Count) usuários..." -ForegroundColor Cyan

foreach ($usuario in $todosUsuarios) {
    if ($contasIgnoradas -contains $usuario.UserPrincipalName) {
        Write-Host "Ignorado: $($usuario.UserPrincipalName)" -ForegroundColor DarkGray
        continue
    }

    try {
        Revoke-MgUserSignInSession -UserId $usuario.Id | Out-Null
        Write-Host "Sucesso: $($usuario.UserPrincipalName)" -ForegroundColor Green
    }
    catch {
        Write-Host "Falha: $($usuario.UserPrincipalName). Erro: $($_.Exception.Message)" -ForegroundColor Red
    }
}

Disconnect-MgGraph
```
