# Habilitar Arquivamento Automático Expandido no Exchange

> **Caminho:** `modern-working/exchange-email/arquivamento-expandido.md`  
> **Objetivo:** Ativar o recurso *Auto-Expanding Archive* no Exchange Online para permitir que o arquivo morto cresça além de 50 GB (até 1,5 TB).

---

## 1. Pré-requisitos

- Caixa de correio com licenciamento compatível com *Exchange Online Archiving* (ex.: E3/E5 ou Plano 2).
- Arquivamento básico (*In-Place Archive*) já ativo na caixa.
- Módulo `ExchangeOnlineManagement` no PowerShell.

---

## 2. Passo a passo

```powershell
# 1. Conectar ao Exchange Online
Connect-ExchangeOnline -UserPrincipalName "admin@seudominio.com.br"

# 2. Habilitar o arquivamento automático expandido para uma caixa específica
Enable-Mailbox "usuario@seudominio.com.br" -AutoExpandingArchive

# 3. Validar se o recurso foi ativado
Get-Mailbox "usuario@seudominio.com.br" |
    Select-Object DisplayName, ArchiveStatus, AutoExpandingArchiveEnabled
```

---

## 3. Validação e Pontos de Atenção

1. O campo `AutoExpandingArchiveEnabled` deve retornar `True`.
2. Uma vez habilitado o arquivamento expandido, ele não pode ser desativado na caixa de correio.
