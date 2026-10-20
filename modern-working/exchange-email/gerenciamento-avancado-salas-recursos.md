# Gerenciamento Avançado de Salas e Recursos no Exchange Online

## 1. Conceito: Visibilidade vs. Acesso

Ao criar uma _Room Mailbox_ no M365, o comportamento padrão é:

- **GAL (Global Address List):** A sala aparece imediatamente para todos na busca.
    
- **Calendário Pessoal:** A sala **não** aparece automaticamente na barra lateral ("Meus Calendários") dos usuários. Eles precisam adicionar manualmente se quiserem monitorar a grade.
    
- **Assistente de Agendamento:** Para aparecer aqui de forma agrupada e organizada (sem precisar digitar o e-mail exato), a sala precisa pertencer a uma **Room List**.
    

## 2. Permissões e Delegação

Diferente de caixas compartilhadas, salas não têm "membros". Elas têm configurações de processamento.

### A. Sala Self-Service (Padrão)

Qualquer um convida, o Exchange aceita/recusa baseado na disponibilidade.

- **Configuração:**
    
    - `AutomateProcessing`: `AutoAccept`
        
    - `ResourceDelegates`: (Vazio/0)
        

### B. Sala Moderada (Com Aprovação)

O usuário convida, a sala fica como "Provisória" e um humano (delegado) precisa aprovar.

- **Configuração:**
    
    - Adicionar usuário em **Delegates** (Representantes) no Exchange Admin Center.
        
    - Isso desativa o `AutoAccept` automaticamente.
        

### C. Sala Restrita (Diretoria)

Apenas um grupo específico pode reservar; outros recebem recusa automática.

- **Configuração (Via PowerShell):**
```powershell
    Set-CalendarProcessing -Identity "SalaDiretoria" -AllBookInPolicy $false -BookInPolicy "GrupoDeSeguranca_Diretoria"
    ```
    

---

## 3. O Problema: "Sala não aparece no Assistente de Agendamento"

**Sintoma:** A sala existe, recebe e-mails, mas não aparece nas listas de seleção rápida do Outlook (Room Finder) ou nas sugestões do Assistente de Agendamento.

**Causa:** A sala não faz parte de uma **Room List** (Lista de Salas). O Exchange Online não adiciona salas às listas automaticamente.

### Solução (Via PowerShell)

É necessário criar ou usar um Grupo de Distribuição especial com o atributo `-RoomList`.

#### Passo 1: Conectar e Verificar Listas Existentes
```powershell
Connect-ExchangeOnline

# Listar apenas grupos que são Room Lists
Get-DistributionGroup -ResultSize Unlimited -Filter "RecipientTypeDetails -eq 'RoomList'"
```

#### Passo 2: Analisar o Retorno

- **Se retornar vazio:** É preciso criar uma lista nova.
```powershell
    New-DistributionGroup -Name "Salas Gerais" -RoomList
    ```
    
- **Se retornar listas (ex: "Salas Matriz"):** Verifique qual está em uso.
```powershell
    Get-DistributionGroupMember -Identity "Salas Matriz"
    ```
    

#### Passo 3: Adicionar a Sala à Lista
```powershell
Add-DistributionGroupMember -Identity "NomeDaLista" -Member "sala.reuniao@seudominio.com.br"
```

> **⚠️ Importante:**
> 
> - **Outlook Web (OWA):** Atualização quase imediata.
>     
> - **Outlook Desktop:** Depende da atualização da OAB (Offline Address Book). Pode levar **24 a 48 horas** para aparecer no dropdown.
>     

---

## 4. Scripts Úteis de Manutenção

### Verificar configurações de processamento da sala
```powershell
Get-CalendarProcessing -Identity "email-da-sala" | Format-List
```

### Configurar Sala Padrão (Limpa e Automática)

Garante que aceita automático, remove o assunto original (privacidade) e coloca o nome do organizador no lugar.
```powershell
Set-CalendarProcessing -Identity "email-da-sala" -AutomateProcessing AutoAccept -DeleteSubject $true -AddOrganizerToSubject $true
```

---
