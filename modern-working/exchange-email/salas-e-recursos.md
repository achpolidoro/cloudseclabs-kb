# Gerenciamento Avançado de Salas e Recursos no Exchange Online

> **Caminho:** `modern-working/exchange-email/salas-e-recursos.md`  
> **Objetivo:** Configurar políticas de reserva automática/moderada de salas de reunião e publicá-las no Assistente de Agendamento (*Room Finder*) usando *Room Lists*.

---

## 1. Pré-requisitos

- Módulo `ExchangeOnlineManagement` no PowerShell.
- Caixas do tipo `RoomMailbox` criadas no Exchange Online.

---

## 2. Passo a passo

### 2.1 Criar Lista de Salas (Room List) para o Assistente do Outlook

Por padrão, salas soltas não aparecem no seletor de locais do Outlook até serem incluídas em um grupo com o atributo `-RoomList`:

```powershell
Connect-ExchangeOnline

# 1. Verificar listas de salas existentes
Get-DistributionGroup -ResultSize Unlimited -Filter "RecipientTypeDetails -eq 'RoomList'"

# 2. Criar uma nova lista de salas (se necessário)
New-DistributionGroup -Name "Salas Matriz" -PrimarySmtpAddress "salas.matriz@seudominio.com.br" -RoomList

# 3. Adicionar a sala à lista
Add-DistributionGroupMember -Identity "Salas Matriz" -Member "sala.reuniao01@seudominio.com.br"
```

### 2.2 Políticas de Reserva (AutoAccept vs. Restrita a Grupo)

```powershell
# Opção A: Reserva automática limpa (mostra o nome do organizador no calendário)
Set-CalendarProcessing -Identity "sala.reuniao01@seudominio.com.br" `
    -AutomateProcessing AutoAccept `
    -DeleteSubject $true `
    -AddOrganizerToSubject $true

# Opção B: Sala restrita apenas a um grupo específico (ex.: Diretoria)
Set-CalendarProcessing -Identity "sala.diretoria@seudominio.com.br" `
    -AllBookInPolicy $false `
    -BookInPolicy "grupo.diretoria@seudominio.com.br"
```

---

## 3. Validação e Pontos de Atenção

1. Confira os parâmetros aplicados na sala com:
```powershell
Get-CalendarProcessing -Identity "sala.reuniao01@seudominio.com.br" | Format-List
```
2. No **Outlook Web (OWA)** a lista de salas aparece em poucos minutos; no **Outlook Desktop** pode levar de 24 a 48 horas devido à atualização do catálogo offline (OAB).
