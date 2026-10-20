# Execução de Assessment M365 via PowerShell e Microsoft Graph

## 0. Antes de tudo

- [ ] Autorização do AMBIENTE_LAB registrada (chamado ou proposta)
- [ ] Confirmado com o responsável técnico o período de coleta (padrão 30 dias)
- [ ] Definido onde as evidências vão morar (ACL restrita, **fora do OneDrive**)

> ⚠️ Os JSON de saída contêm UPN, IP, país de sign-in e dados de contas
> privilegiadas. É dado do AMBIENTE_LAB. Trate como confidencial e defina retenção.

---

## 1. Pedido ao AMBIENTE_LAB

Manda esse texto. Evita o vai-e-volta de permissão faltando.

> Para executar o assessment, preciso que seja criado um App Registration no
> tenant, com as permissões abaixo. Todas são **somente leitura** e do tipo
> **Application permissions** (não delegated).
>
> **Microsoft Graph — Application permissions:**
>
> 1. Organization.Read.All
> 2. Directory.Read.All
> 3. User.Read.All
> 4. Group.Read.All
> 5. Team.ReadBasic.All
> 6. Policy.Read.All
> 7. SecurityEvents.Read.All
> 8. AuditLog.Read.All
> 9. UserAuthenticationMethod.Read.All
> 10. Device.Read.All
> 11. DeviceManagementManagedDevices.Read.All
> 12. Application.Read.All
> 13. RoleManagement.Read.Directory
> 14. RoleEligibilitySchedule.Read.Directory
> 15. IdentityRiskyUser.Read.All
> 16. IdentityRiskEvent.Read.All
>
> Depois de adicionar, clicar em **Grant admin consent** — todas precisam ficar
> com status concedido (check verde).
>
> Em seguida, gerar um **client secret** e me enviar:
> - Directory (tenant) ID
> - Application (client) ID
> - O **Value** do secret (aparece só uma vez)

**Ponto de atenção:** as duas de Identity Protection (15 e 16) são as que
costumam ficar de fora. Se faltar, o coletor `IdentityProtection` dá 403.

---

## 2. Preparar a pasta do AMBIENTE_LAB

Uma pasta por AMBIENTE_LAB. Sem exceção — evita misturar evidência.

```powershell
# copia a CLI para uma pasta nova
Copy-Item "C:\Tools\assessmentm365-main" "C:\Assessments\AMBIENTE_LAB" -Recurse

# remove output herdado da cópia
Remove-Item "C:\Assessments\AMBIENTE_LAB\output" -Recurse -Force -ErrorAction SilentlyContinue
```

---

## 3. Execução

### 3.1 Preparo (só na primeira vez por máquina)

```powershell
cd "C:\Assessments\AMBIENTE_LAB"
./scripts/install-dependencies.ps1
```

### 3.2 Validação sem autenticar

```powershell
./Invoke-M365Assessment.ps1 -ValidateOnly   # espera: valid : True
./Invoke-M365Assessment.ps1 -DryRun         # lista os 14 coletores
```

### 3.3 Coleta

```powershell
$secret = Read-Host "Cole o secret" -AsSecureString
$secret.Length   # tem que ser > 0

./Invoke-M365Assessment.ps1 `
  -AuthenticationMode AppOnly `
  -TenantId "<TENANT_ID>" `
  -ClientId "<APP_ID>" `
  -ClientSecret $secret `
  -StartDate (Get-Date).AddDays(-30)
```

> Nada aparece na tela ao colar o secret. É o `-AsSecureString`, é normal.

### 3.4 Conferir

```powershell
cat ./output/summary.json

# detalhe das falhas
(Get-Content ./output/summary.json -Raw | ConvertFrom-Json).results |
  Where-Object status -eq 'failed' |
  Select-Object name, error | Format-List
```

**Critério de aceite:** `totals.failed = 0` e `totals.collectors = 14`.

### 3.5 Encerrar

```powershell
Move-Item ./output "C:\Evidencias\AMBIENTE_LAB\output_$(Get-Date -f yyyy-MM-dd)"
```

Fecha o PowerShell. Isso descarta o `$secret` da memória.

---

## 4. Regra de ouro do retrabalho

> **Mudou permissão no portal? Fecha o PowerShell, espera 5 minutos, abre de novo.**

O token fica em cache na sessão. Rodar na mesma janela reaproveita o token
antigo, sem a permissão nova — e você acha que não resolveu. Perdi tempo com
isso.

---

## 5. Erros e tratativas

| Erro | Causa | O que fazer |
|---|---|---|
| `Parameter set cannot be resolved` | ClientId indo junto do ClientSecretCredential | Usar a CLI com a correção no `Graph.ps1` |
| Falha de autenticação Graph | Secret errado/expirado, ou consent não concedido | Conferir check verde no portal |
| Coletor com `403 Forbidden` | Permissão faltando **ou** licença insuficiente | Adicionar permissão + consent. Persistindo → licença |
| Falha após adicionar permissão | Token em cache / propagação | Fechar janela, esperar 5 min |
| `DomainAuthentication` skipped | Sem `Resolve-DnsName` nem `dig` | Rodar em host Windows |

### Dependência de licença

| Coletor | Exige |
|---|---|
| ConditionalAccessSecurityDefaults | Entra ID P1 |
| SignInsLegacyAuth | **P1** (sign-in logs não existem na API no Free) |
| IdentityProtection | **Entra ID P2** |
| PimEligibility | Entra ID P2 |
| IntuneCompliance | Licença Intune |

**Checar a licença antes de rodar.** Em tenant Free, a parte de identidade —
que é a mais vendável — simplesmente não sai. Ajustar expectativa do AMBIENTE_LAB
antes, não depois.

---

## 6. Armadilhas de interpretação

Três coisas que o script reporta de um jeito que **não** deve ir cru pro relatório:

**DKIM "não encontrado"** — ele só testa `selector1` e `selector2`. Se o AMBIENTE_LAB
usa selector customizado, é falso negativo. **Confirmar no portal.**

**"Nenhuma conta break-glass identificada"** — só quer dizer que o
`config/assessment.json` está com o bloco `breakGlass` vazio. Não é a mesma
coisa que "não existe". Levantar com o AMBIENTE_LAB e preencher:

```json
"breakGlass": {
  "userObjectIds": ["<guid>", "<guid>"],
  "accountNamePattern": "^(break|emergency|bg)-"
}
```

**`roleDisplayName: null`** — atribuição privilegiada cuja role o script não
conseguiu nomear (provável role customizada). Não deixa buraco no relatório:
olha no portal qual é.

---

## 7. O que este assessment NÃO cobre

Com client secret ficam de fora:

- **Exchange Online** — forwarding externo, inbox rules de encaminhamento,
  auditoria de mailbox, litigation hold, retenção
- **SharePoint Online** — sharing capability por site, usuários externos
- **Purview / DLP**

Motivo: `Connect-ExchangeOnline` app-only **não aceita client secret** (limitação
da Microsoft, não do script), e o PnP com secret dependia do ACS, que foi
aposentado. Os dois exigem **certificado**.

### Coleta manual como paliativo (Global Reader)

```powershell
Connect-ExchangeOnline -UserPrincipalName eu@AMBIENTE_LAB.com

Get-EXOMailbox -ResultSize Unlimited `
  -Properties ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward,AuditEnabled,LitigationHoldEnabled |
  Where-Object { $_.ForwardingAddress -or $_.ForwardingSmtpAddress } |
  Export-Csv .\forwarding.csv -NoTypeInformation -Encoding UTF8

Get-OrganizationConfig | Select-Object AuditDisabled
```

- `Get-InboxRule -Mailbox <outro>` pode falhar só com Global Reader → pedir role
  **View-Only Recipients** no Exchange admin center
- SharePoint: usar o **admin center pela web** (Sites ativos → exportar CSV).
  O PnP exige `-ClientId` de app com permissões **delegadas**

---

## 8. Evolução — migrar para certificado

Quando der, vale. Destrava Exchange e SharePoint **sem alterar o código** — o
contexto dos plugins já carrega o `certificateThumbprint`.

```powershell
$cert = New-SelfSignedCertificate -Subject "CN=Assessment365-AMBIENTE_LAB" `
  -CertStoreLocation "Cert:\CurrentUser\My" `
  -KeyExportPolicy Exportable -KeySpec Signature -NotAfter (Get-Date).AddYears(1)
$cert.Thumbprint

Export-Certificate -Cert $cert -FilePath "$HOME\Desktop\Assessment365-AMBIENTE_LAB.cer"
```

Sobe o `.cer` em `Certificates & secrets → Certificates → Upload certificate`.
Não precisa de consent novo — é só uma credencial a mais no app.

**Extras necessários:** `Exchange.ManageAsApp` (Office 365 Exchange Online),
`Sites.Read.All` (Graph), e a role **View-Only Organization Management** no
Exchange admin center para o service principal.

Depois: `./scripts/install-dependencies.ps1 -IncludeSharePoint -IncludeExchange`,
habilitar os plugins no config e trocar `-ClientSecret` por
`-CertificateThumbprint`.

**Um certificado por AMBIENTE_LAB.** Nunca reaproveitar.

---

## 9. Encerramento

- [ ] Evidências no repositório seguro, com data no nome
- [ ] Chamado atualizado: data, período, tenant, local das evidências
- [ ] Relatório entregue
- [ ] **Secret revogado ou App Registration excluído** pelo AMBIENTE_LAB
- [ ] Cópias locais do secret descartadas
