# Revogação Global de Sessões e Tokens no Entra ID

> **Caminho:** `modern-working/identidade-sso/revogar-sessoes-tokens.md`  
> **Objetivo:** Invalidar todos os *Refresh Tokens* ativos dos usuários do diretório via PowerShell 7 e Microsoft Graph (forçando novo login), preservando contas administrativas de emergência (*break-glass*).

---

## 1. Pré-requisitos

- **PowerShell 7+** com módulo `Microsoft.Graph` instalado.
- Conta com escopos `User.ReadWrite.All` e `Directory.ReadWrite.All`.

---

## 2. Passo a passo

Execute o bloco abaixo no PowerShell 7 ajustando o domínio do tenant e a lista de exclusão:

```powershell
# 1. Conectar ao Microsoft Graph
Connect-MgGraph -TenantId "seudominio.onmicrosoft.com" `
                -Scopes "User.ReadWrite.All", "Directory.ReadWrite.All"

# 2. Lista de exclusão (Contas Break-Glass que não devem ser desconectadas)
$contasIgnoradas = @(
    "admin.emergencia@seudominio.onmicrosoft.com"
)

# 3. Revogar sessões de todos os demais usuários
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

---

## 3. Validação e Pontos de Atenção

1. Verifique nos logs de auditoria do Entra ID (`Audit logs`) o evento `StsRefreshTokenValidFrom` atualizado para as contas processadas.
2. Sessões baseadas em tokens de acesso vigentes podem levar até 1 hora para expirar caso a Avaliação Contínua de Acesso (CAE) não esteja ativa.
