# Substituição de Assinaturas de E-mail em Massa via PowerShell

## Objetivo

Substituir a assinatura de e-mail de todos os colaboradores do tenant Microsoft 365 de uma só vez, via PowerShell.

---

## Contexto — por que o script antigo parou

O script anterior conectava ao Exchange Online usando **Remote PowerShell com Basic Auth** (`New-PSSession` + `-Authentication Basic` + `ps.outlook.com`). A Microsoft aposentou o Basic Auth e removeu o Remote PowerShell do Exchange Online. O script falha na conexão e nunca chega a alterar assinatura alguma.

A lógica de assinatura (`Set-MailboxMessageConfiguration`) continua válida. **Só a conexão precisa ser trocada.**

Existe um segundo ponto: a Microsoft migrou as assinaturas para o modelo _roaming signatures_. Quando esse modelo está ativo, o Outlook ignora o campo alterado pelo script. Por isso o Passo 3 é obrigatório.

---

## Pré-requisitos

- Conta com papel **Exchange Administrator** ou **Organization Management ou Global Administrator**
- **PowerShell 7** (não usar ISE)
- Assinatura em HTML salva no repositório
- Logo hospedado em **URL pública** (não usar imagem local nem base64)

---

## Passo 1 — Instalar o módulo

```powershell
Install-Module ExchangeOnlineManagement -Scope CurrentUser -Force
```

Se der erro de comando já existente, adicione `-AllowClobber`.

---

## Passo 2 — Conectar

```powershell
Connect-ExchangeOnline
```

Abre janela de login no navegador. Confirme com:

```powershell
Get-ConnectionInformation
```

Se retornar vazio, não conectou.

---

## Passo 3 — Ativar o modo legado de assinatura

> ⚠️ Alteração **global do tenant**. Valide o impacto antes de aplicar em produção.

```powershell
Set-OrganizationConfig -PostponeRoamingSignaturesUntilLater $true
```

Confirmar:

```powershell
Get-OrganizationConfig | fl PostponeRoamingSignaturesUntilLater
```

Deve retornar `True`.

**Aguardar de 10 a 60 minutos para propagação.** Não conclua que falhou antes disso.

---

## Passo 4 — Preparar os arquivos

Criar a pasta `C:\temp\assinaturas\` com um arquivo HTML por usuário, nomeado pelo e-mail:

```
C:\temp\assinaturas\joao.silva@empresa.com.br.html
C:\temp\assinaturas\maria.souza@empresa.com.br.html
```

O nome do arquivo (sem `.html`) é usado como identidade da caixa de correio.

---

## Passo 5 — Teste em uma caixa

Antes de rodar em massa, validar em **uma** conta:

```powershell
Set-MailboxMessageConfiguration usuario@empresa.com.br `
  -SignatureHtml (Get-Content 'C:\temp\assinaturas\usuario@empresa.com.br.html' -Raw) `
  -AutoAddSignature $true -AutoAddSignatureOnReply $true -ErrorAction Stop
```

Verificar a gravação:

```powershell
Get-MailboxMessageConfiguration usuario@empresa.com.br | fl SignatureHtml, AutoAddSignature
```

Abrir o **Outlook Web em janela anônima** → Nova mensagem → confirmar que a assinatura aparece.

> Testar também em uma caixa que **já possua** assinatura configurada.

---

## Passo 6 — Aplicar em massa

```powershell
$pasta = 'C:\temp\assinaturas\'
$log = @()

foreach ($item in Get-ChildItem -Path $pasta -Filter *.html) {
    $usuario = $item.BaseName
    try {
        Set-MailboxMessageConfiguration -Identity $usuario `
            -SignatureHtml (Get-Content $item.FullName -Raw) `
            -AutoAddSignature $true -AutoAddSignatureOnReply $true -ErrorAction Stop
        $status = 'OK'; $erro = ''
    }
    catch { $status = 'ERRO'; $erro = $_.Exception.Message }

    Write-Host "$status - $usuario"
    $log += [pscustomobject]@{ Usuario = $usuario; Status = $status; Erro = $erro }
    Start-Sleep -Milliseconds 300
}

$log | Export-Csv .\log-assinaturas.csv -NoTypeInformation -Encoding UTF8
```

> Em ambientes com mais de 200 caixas, executar em lotes.

---

## Passo 7 — Encerrar e documentar

```powershell
Disconnect-ExchangeOnline -Confirm:$false
```

Conferir o arquivo `log-assinaturas.csv` gerado. Reexecutar apenas nas linhas com `Status = ERRO`.

---

## Reversão

```powershell
Set-OrganizationConfig -PostponeRoamingSignaturesUntilLater $false
```

A alteração de assinatura em si é sobrescrita por nova execução do script.

---

## Pontos de atenção

|Item|Observação|
|---|---|
|Assinaturas múltiplas|O modo legado desativa assinaturas nomeadas. Usuários voltam a ter uma só.|
|Sincronização entre dispositivos|No Outlook desktop a assinatura volta a ser local, sem sincronizar.|
|Novo Outlook|Depende do modelo roaming. Validar comportamento antes de implantar.|
|Solução temporária|O parâmetro é de **adiamento**. A Microsoft deve removê-lo no futuro.|
|Usuários novos|Não recebem assinatura automaticamente. Reexecutar o script na admissão.|
|Múltiplos domínios|Usar uma pasta e um template por domínio, executando o Passo 6 para cada.|

---
