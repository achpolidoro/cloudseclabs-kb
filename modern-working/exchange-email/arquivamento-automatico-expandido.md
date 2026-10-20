# Habilitar Arquivamento Automático Expandido no Exchange Online

> **Objetivo:** Permitir que a caixa de arquivo morto (*In-Place Archive*) cresça além do limite inicial de 50 GB (até 1,5 TB) de forma automática.

## 1. Pré-requisitos

- Permissão de **Exchange Administrator**
- Módulo `ExchangeOnlineManagement` instalado no PowerShell

## 2. Passo a passo via PowerShell

1. Conecte-se ao Exchange Online:
```powershell
Connect-ExchangeOnline -UserPrincipalName "admin@seudominio.com.br"
```

2. Habilite o arquivamento automático expandido para a caixa desejada:
```powershell
Enable-Mailbox "usuario@seudominio.com.br" -AutoExpandingArchive
```

3. Verifique se a configuração foi aplicada:
```powershell
Get-Mailbox "usuario@seudominio.com.br" | Select-Object DisplayName, ArchiveStatus, AutoExpandingArchiveEnabled
```

## 3. Referência Oficial
- `https://learn.microsoft.com/pt-br/purview/enable-autoexpanding-archiving`
