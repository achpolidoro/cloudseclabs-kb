# Definir Proprietário (Owner) em Equipes do Teams via PowerShell

> **Objetivo:** Adicionar um administrador ou gestor como proprietário (`Owner`) em todas as equipes existentes no tenant via módulo `MicrosoftTeams`.

## 1. Instalação e Conexão

```powershell
Install-Module MicrosoftTeams -Scope CurrentUser
Connect-MicrosoftTeams
```

## 2. Aplicar Proprietário em Lote

```powershell
# Conta que será adicionada como proprietária
$usuario = "admin.gestao@seudominio.com.br"

# Listar todas as equipes do ambiente
$equipes = Get-Team

foreach ($equipe in $equipes) {
    Write-Host "Adicionando $usuario como proprietário na equipe: $($equipe.DisplayName)" -ForegroundColor Yellow
    Add-TeamUser -GroupId $equipe.GroupId -User $usuario -Role Owner
}
```
