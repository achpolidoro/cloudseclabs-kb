# Definir Proprietário (Owner) em Equipes do Teams em Lote

> **Caminho:** `modern-working/sharepoint-teams/definir-proprietario-teams.md`  
> **Objetivo:** Adicionar uma conta administrativa ou de governança como proprietária (`Owner`) em todas as equipes do Microsoft Teams via PowerShell.

---

## 1. Pré-requisitos

- Módulo `MicrosoftTeams` instalado no PowerShell.
- Permissão de **Teams Administrator** ou **Global Administrator**.

---

## 2. Passo a passo

```powershell
# 1. Instalar módulo e autenticar no Microsoft Teams
Install-Module MicrosoftTeams -Scope CurrentUser
Connect-MicrosoftTeams

# 2. Definir a conta que receberá permissão de Owner
$usuario = "admin.governanca@seudominio.com.br"

# 3. Listar todas as equipes e adicionar o proprietário
$equipes = Get-Team

foreach ($equipe in $equipes) {
    Write-Host "Adicionando $usuario como Owner em: $($equipe.DisplayName)" -ForegroundColor Cyan
    Add-TeamUser -GroupId $equipe.GroupId -User $usuario -Role Owner
}
```

---

## 3. Validação e Pontos de Atenção

1. Para auditar equipes órfãs (sem nenhum proprietário ativo), execute:
```powershell
Get-Team | ForEach-Object {
    $owners = Get-TeamUser -GroupId $_.GroupId -Role Owner
    if (-not $owners) { Write-Host "Equipe sem Owner: $($_.DisplayName)" -ForegroundColor Yellow }
}
```
