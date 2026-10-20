# Checklist de Verificações e Coleta de Evidências (Postura M365 e Entra ID)



**Regra:** o print é demonstrativo. O dado completo deve ser preservado em CSV/XLSX e nos JSON da CLI.

**Padrão para todos os prints:** manter a data/hora visível e o nome do tenant na tela quando possível. Registrar também o portal utilizado, o filtro aplicado e o período consultado.

**Portais principais:**

- Microsoft Entra admin center: `https://entra.microsoft.com`

- Microsoft 365 admin center: `https://admin.microsoft.com`

- Exchange admin center: `https://admin.exchange.microsoft.com`

- Microsoft Defender portal: `https://security.microsoft.com`

- SharePoint admin center: `https://admin.microsoft.com/sharepoint`

- Teams admin center: `https://admin.teams.microsoft.com`

- Intune admin center: `https://intune.microsoft.com`

- Microsoft Purview: `https://purview.microsoft.com`

---

## Anexo B.1 — Identidade / Entra ID

### B.1.1 Visão geral do tenant

- [ ] **Visão geral do tenant — nome, ID e licenças ativas**
  - Caminho: `Entra ID` → `Overview`
  - Registrar: nome do tenant, Tenant ID, domínio primário, localização, edição/licenciamento e estado do diretório.
  - Licenças: `Microsoft 365 admin center` → `Billing` → `Your products` ou `Licenses` → `All products`.

### B.1.2 Security Defaults

- [ ] **Propriedades → Security Defaults — ligado/desligado**
  - Caminho: `Entra ID` → `Overview` → `Properties` → `Manage security defaults`.
  - Registrar o estado e, se possível, a data da última alteração.

### B.1.3 Conditional Access

- [ ] **Lista de políticas de Acesso Condicional — nome e estado**
  - Caminho: `Entra ID` → `Protection` → `Conditional Access` → `Policies`.
  - Registrar: nome, estado, data de alteração e plataforma/escopo aparente.

- [ ] **Detalhe de cada CA relevante — usuários, aplicativos, condições e controles**
  - Caminho: `Entra ID` → `Protection` → `Conditional Access` → `Policies` → selecionar a política.
    - Verificar as seções:
      - `Users and groups`
      - `Target resources` ou `Cloud apps or actions`
      - `Conditions`
      - `Access controls` → `Grant` e `Session`
      - `Enable policy`
      - Exclusões e modo somente relatório, quando aplicável.

### B.1.4 Métodos de autenticação e SSPR

- [ ] **Métodos de autenticação — política e métodos habilitados**
  - Caminho: `Entra ID` → `Protection` → `Authentication methods` → `Policies`.
  - Verificar: Microsoft Authenticator, FIDO2/passkeys, SMS, voz, Temporary Access Pass, email e métodos legados.
  - Complemento: `Entra ID` → `Protection` → `Authentication methods` → `Activity`.

- [ ] **SSPR — propriedades e público habilitado**
  - Caminho: `Entra ID` → `Protection` → `Password reset` → `Properties`.
  - Verificar: habilitado para nenhum, selecionado ou todos; grupos incluídos/excluídos.
  - Demais configurações: `Password reset` → `Authentication methods`, `Registration` e `Notifications`.

- [ ] **Usuários sem MFA registrado**
  - Caminho: `Entra ID` → `Users` → `All users` → `Per-user MFA` ou relatório de métodos de autenticação.
  - Relatório: `Entra ID` → `Protection` → `Authentication methods` → `Activity` → `User registration details`.
  - Filtrar usuários sem método forte registrado e preservar o relatório exportado.

### B.1.5 Configurações de usuários e colaboração externa

- [ ] **Configurações de usuário — quem pode registrar aplicativos e criar tenants**
  - Registro de apps: `Entra ID` → `App registrations` → `User settings` → `Users can register applications`.
  - Criação de tenants: `Entra ID` → `Users` → `User settings` → `Tenant creation`.
  - Registrar também as demais opções relevantes de `Users` → `User settings`.

- [ ] **Configurações de colaboração externa — quem pode convidar convidados**
  - Caminho: `Entra ID` → `External Identities` → `External collaboration settings`.
  - Verificar: `Guest invite settings`, `Guest user access restrictions` e `Collaboration restrictions`.
  - Registrar domínios permitidos/bloqueados, se configurados.

- [ ] **Lista de convidados e último sign-in**
  - Caminho: `Entra ID` → `Users` → `All users`.
  - Filtrar: `User type = Guest`.
  - Verificar/exportar: nome, UPN, origem, estado, data de criação e `Last interactive sign-in`.
  - Detalhe individual: selecionar usuário → `Overview` → `Sign-in activity`.
  - Logs: `Entra ID` → `Monitoring & health` → `Sign-in logs`.

---

## Anexo B.2 — Privilégios e PIM

- [ ] **Funções atribuídas: Global Admin — lista completa**
  - Caminho: `Entra ID` → `Roles & admins` → `Global Administrator` → `Assignments`.
  - Conferir `Active assignments`, `Eligible assignments` e, quando disponível, `Expired assignments`.

- [ ] **Demais funções privilegiadas com atribuição permanente**
  - Caminho: `Entra ID` → `Roles & admins` → selecionar cada função → `Assignments`.
  - Identificar `Permanent`/`Permanently assigned` e separar de atribuições elegíveis ou com prazo.
  - Priorizar: Privileged Role Administrator, Security Administrator, Exchange Administrator, SharePoint Administrator, User Administrator, Authentication Administrator, Conditional Access Administrator, Application Administrator e Cloud Application Administrator.

- [ ] **PIM → funções do Entra → atribuições elegíveis versus ativas**
  - Caminho: `Identity Governance` → `Privileged Identity Management` → `Microsoft Entra roles` → `Assignments`.
  - Preservar as visões `Active`, `Eligible` e `Expired`, se disponíveis.

- [ ] **PIM → configurações da função Global Admin**
  - Caminho: `Identity Governance` → `Privileged Identity Management` → `Microsoft Entra roles` → `Roles` → `Global Administrator` → `Role settings`.
  - Verificar: MFA, justificativa, ticket/motivo, aprovação, aprovadores, duração máxima, ativação permanente e notificações.

- [ ] **Contas break-glass — existência, exclusão em CA e MFA**
  - Existência: `Entra ID` → `Users` → `All users`.
  - Funções e estado: selecionar conta → `Overview` e `Assigned roles`.
  - MFA/métodos: selecionar conta → `Authentication`.
  - Exclusões de CA: `Entra ID` → `Protection` → `Conditional Access` → `Policies` → abrir cada política → `Users and groups` → `Exclude`.
  - Uso da conta: `Entra ID` → `Monitoring & health` → `Sign-in logs` → filtrar pelo usuário.

---

## Anexo B.3 — Proteção de identidade

- [ ] **Identity Protection → políticas de risco de usuário e de entrada**
  - Caminho: `Entra ID` → `Protection` → `Identity Protection` → `Risk policies`.
  - Complemento: `Entra ID` → `Protection` → `Conditional Access` → `Policies` → filtrar políticas com `User risk` ou `Sign-in risk`.
  - Registrar nível de risco, ação, MFA, alteração de senha, bloqueio, exclusões e modo da política.

- [ ] **Relatório de usuários em risco**
  - Caminho: `Entra ID` → `Protection` → `Identity Protection` → `Risky users`.
  - Registrar usuário, nível, estado, data da detecção, última atualização e ação de correção.

- [ ] **Relatório de detecções de risco**
  - Caminho: `Entra ID` → `Protection` → `Identity Protection` → `Risk detections`.
  - Registrar tipo, usuário, IP, localização, aplicativo, client app, data/hora, nível e status.

---

## Anexo B.4 — Aplicativos

- [ ] **Enterprise apps — lista com consentimento concedido**
  - Caminho: `Entra ID` → `Enterprise applications` → `All applications`.
  - Selecionar aplicativo → `Permissions` → `Admin consent`.
  - Verificar permissões concedidas, usuários/grupos atribuídos, publisher, data de criação e estado.
  - Complemento: `Entra ID` → `Enterprise applications` → `Consent and permissions`.

- [ ] **Configurações de consentimento do usuário — pode consentir sozinho?**
  - Caminho: `Entra ID` → `Enterprise applications` → `Consent and permissions` → `User consent settings`.
  - Verificar quem pode consentir, tipos de aplicativo, permissões de baixo impacto e fluxo de solicitação administrativa.

- [ ] **Apps sem publisher verificado**
  - Caminho: `Entra ID` → `Enterprise applications` → `All applications`.
  - Verificar coluna `Publisher verified`.
  - Registros: `Entra ID` → `App registrations` → `All applications`; conferir publisher domain e status de verificação.

- [ ] **Registros de app com segredo/certificado vencido ou de longa duração**
  - Caminho: `Entra ID` → `App registrations` → `All applications` → selecionar aplicativo → `Manage` → `Certificates & secrets`.
  - Registrar segredos/certificados expirados, validade longa, ausência de rotação e proprietários.

---

## Anexo B.5 — Exchange Online

- [ ] **`Get-OrganizationConfig` ** → ** `AuditDisabled`**
  - Exchange Online PowerShell:

```
Get-OrganizationConfig | Format-List AuditDisabled
```

- Complemento: `Microsoft Purview` → `Audit`; verificar se a auditoria está disponível e com eventos recentes.

- [ ] **Regras de fluxo de e-mail — transport rules**
  - Portal: `Exchange admin center` → `Mail flow` → `Rules`.
  - PowerShell:

```
Get-TransportRule
```

- Destacar regras de encaminhamento, redirecionamento, cópia externa, bypass de segurança, alteração de remetente/destinatário e exceções amplas.

- [ ] **Encaminhamento externo — política e caixas com forward configurado**
  - Política: `Microsoft Defender portal` → `Email & collaboration` → `Policies & rules` → `Threat policies` → `Anti-spam policies` → `Outbound spam filter policy` → `Automatic forwarding`.
  - Caixa individual: `Exchange admin center` → `Recipients` → `Mailboxes` → selecionar caixa → `Mail flow settings` → `Email forwarding`.
  - PowerShell:

```
Get-Mailbox -ResultSize Unlimited |
  Select-Object DisplayName,PrimarySmtpAddress,ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward
```

- [ ] **Autenticação moderna e protocolos legados — POP, IMAP e SMTP AUTH**
  - Configuração geral: `Microsoft 365 admin center` → `Settings` → `Org settings` → `Modern authentication`.
  - Usuário: `Microsoft 365 admin center` → `Users` → `Active users` → selecionar usuário → `Mail` → `Manage email apps`.
  - PowerShell:

```
Get-CASMailbox -ResultSize Unlimited |
  Select-Object DisplayName,PopEnabled,ImapEnabled,SmtpClientAuthenticationDisabled
```

- [ ] **Anti-phishing, anti-spam e anti-malware — políticas em uso**
  - Caminho: `Microsoft Defender portal` → `Email & collaboration` → `Policies & rules` → `Threat policies`.
  - Abrir `Anti-phishing`, `Anti-spam`, `Anti-malware`, `Tenant Allow/Block List` e `Outbound spam filter policy`.
  - Registrar prioridade, escopo, ações e exclusões.

- [ ] **Safe Links / Safe Attachments — se licenciado**
  - Caminho: `Microsoft Defender portal` → `Email & collaboration` → `Policies & rules` → `Threat policies` → `Safe Links` ou `Safe Attachments`.
  - Registrar políticas ativas, escopo, exclusões e ação aplicada.

- [ ] **Quarentena — política de notificação**
  - Mensagens: `Microsoft Defender portal` → `Email & collaboration` → `Review` → `Quarantine`.
  - Política: `Email & collaboration` → `Policies & rules` → `Threat policies` → abrir política correspondente → `Quarantine policy`.
  - Verificar destinatários, frequência e permissões de liberação.

---

## Anexo B.6 — SharePoint e OneDrive

- [ ] **Compartilhamento externo — nível do tenant**
  - Caminho: `SharePoint admin center` → `Policies` → `Sharing`.
  - Verificar separadamente SharePoint e OneDrive, convidados externos, compartilhamento anônimo e domínios permitidos/bloqueados.

- [ ] **Tipo de link padrão e expiração de link anônimo**
  - Caminho: `SharePoint admin center` → `Policies` → `Sharing`.
  - Verificar `Default link type`, `Default link permission`, expiração de links `Anyone`, edição e exigência de autenticação.

- [ ] **Acesso de dispositivo não gerenciado**
  - Caminho: `SharePoint admin center` → `Access control` → `Unmanaged devices`.
  - Complemento: `Entra ID` → `Protection` → `Conditional Access` → `Policies`.
  - Registrar se está bloqueado, limitado ou integralmente permitido.

- [ ] **Sites com compartilhamento mais permissivo que o tenant**
  - Caminho: `SharePoint admin center` → `Active sites` → selecionar site → `Settings` → `More sharing settings`.
  - Comparar com `Policies` → `Sharing`.
  - Conferir também `Active sites` → selecionar site → `Permissions`, links anônimos, convidados e owner ativo.

---

## Anexo B.7 — Teams

- [ ] **Política de acesso externo e de convidado**
  - Acesso externo: `Teams admin center` → `Users` → `External access` ou `Teams` → `Policies` → `External access policies`.
  - Convidado: `Teams admin center` → `Users` → `Guest access` ou `Teams` → `Policies` → `Guest policies`.
  - Registrar domínios, chat, chamadas, reuniões, compartilhamento de tela, criação/edição de canais e aplicativos.

- [ ] **Quem pode criar equipes**
  - Caminho principal: `Microsoft 365 admin center` → `Settings` → `Org settings` → `Microsoft 365 Groups`.
  - Verificar configuração de criação de grupos e grupo de segurança autorizado.
  - Complemento: `Entra ID` → `Groups` → `Group settings`.
  - Registrar a relação entre criação de Microsoft 365 Groups e criação de Teams.

- [ ] **Convidados órfãos — equipes sem owner ativo**
  - Caminho: `Teams admin center` → `Teams` → `Manage teams` → selecionar equipe → `Owners`.
  - Complemento: `Microsoft 365 admin center` → `Teams & groups` → `Active teams & groups`.
  - Validar cada owner em `Entra ID` → `Users` → `All users`: ativo, excluído, bloqueado e último sign-in.

---

## Anexo B.8 — Dispositivos e Intune

- [ ] **Dispositivos registrados no Entra — join type**
  - Caminho: `Entra ID` → `Devices` → `All devices`.
  - Registrar Device ID, join/trust type, owner, sistema operacional e último sign-in aproximado.
  - Tipos: Microsoft Entra joined, Microsoft Entra registered e Hybrid Microsoft Entra joined.
  - Complemento: `Intune admin center` → `Devices` → `All devices`.

- [ ] **Políticas de conformidade — lista e estado**
  - Caminho: `Intune admin center` → `Devices` → `Compliance` → `Policies`.
  - Verificar nome, plataforma, atribuições, estado, data de criação e alteração.
  - Relatórios: `Reports` → `Device compliance` ou `Devices` → `Monitor` → `Compliance`.

- [ ] **Dispositivos não conformes — resumo**
  - Caminho: `Intune admin center` → `Devices` → `Compliance` → `Devices` → filtrar `Noncompliant`.
  - Complemento: `Reports` → `Device compliance`.
  - Registrar total, distribuição por plataforma, motivo e data da última avaliação.

- [ ] **Regras de restrição de registro**
  - Caminho: `Intune admin center` → `Devices` → `Enrollment` → `Enrollment restrictions`.
  - Verificar `Device limit restrictions`, `Platform restrictions`, grupos incluídos/excluídos e prioridade.
  - Complemento: `Devices` → `Enrollment` → `Windows enrollment` e `Device platform restrictions`.

---

## Anexo B.9 — Defender / postura

- [ ] **Secure Score atual e histórico**
  - Caminho: `Microsoft Defender portal` → `Secure score` → `Overview`.
  - Histórico: `Secure score` → `Score analyzer` ou `History`, conforme a interface.
  - Registrar pontuação, percentual, pontuação máxima, tendência e comparação disponível.

- [ ] **Top ações recomendadas não implementadas**
  - Caminho: `Secure score` → `Improvement actions`.
  - Ordenar por pontos, impacto, produto, status e prazo.
  - Registrar ações `To address`, `Planned`, `Resolved`, `Resolved through third party` e `Risk accepted`.

- [ ] **Defender XDR → incidentes abertos — se licenciado**
  - Caminho: `Microsoft Defender portal` → `Incidents & alerts` → `Incidents`.
  - Filtrar `Active` ou `In progress`, severidade e período.
  - Complemento: `Incidents & alerts` → `Alerts`.

---

## Anexo B.10 — Domínio e e-mail autenticado

- [ ] **Domínios verificados no tenant**
  - Microsoft 365: `Microsoft 365 admin center` → `Settings` → `Domains`.
  - Entra: `Entra ID` → `Custom domain names`.
  - Registrar domínio, estado de verificação, serviços associados e domínio padrão.

- [ ] **SPF, DKIM e DMARC de cada domínio — saída do coletor**
  - SPF: consultar o TXT do domínio, normalmente começando por `v=spf1`.
  - DMARC: consultar `_dmarc.exemplo.com`, normalmente começando por `v=DMARC1`.
  - DKIM: consultar `selector1._domainkey.exemplo.com` e `selector2._domainkey.exemplo.com`, ou os seletores fornecidos pelo serviço.
  - Preservar a saída do coletor com domínio, registro completo, política DMARC e observações.

| Domínio | SPF | DKIM DNS | DMARC | Política DMARC | Observações |
| --- | --- | --- | --- | --- | --- |
|  |  |  |  |  |  |

- [ ] **DKIM no portal Defender — habilitado por domínio**
  - Caminho: `Microsoft Defender portal` → `Email & collaboration` → `Policies & rules` → `Threat policies` → `Email authentication settings` → `DKIM`.
  - Verificar por domínio: habilitado, CNAMEs publicados, assinatura ativa e domínios não configurados.

---

## Anexo C — Registros de autenticação

- [ ] **Sign-ins com autenticação legada — filtro por Client app**
  - Caminho: `Entra ID` → `Monitoring & health` → `Sign-in logs`.
  - Filtro: `Client app`; selecionar POP, IMAP, SMTP, Exchange ActiveSync e outros clientes legados disponíveis.
  - Preservar intervalo de datas, total de resultados e exportação.

- [ ] **Coluna ** `errorCode` ** visível — 53003 = bloqueado por CA, não é brecha**
  - Caminho: `Entra ID` → `Monitoring & health` → `Sign-in logs` → `Columns`.
  - Habilitar: `Error code`, `Failure reason`, `Conditional Access`, `Client app`, IP, localização e requisito de autenticação.
  - Interpretar o código 53003 em conjunto com a política de Conditional Access e o motivo da falha; ele normalmente indica bloqueio por CA.

- [ ] **Sign-ins de contas privilegiadas**
  - Caminho: `Entra ID` → `Monitoring & health` → `Sign-in logs`.
  - Filtrar por usuários privilegiados, grupos/funções, período, aplicativo, localização e status.
  - Obter a relação de usuários em `Entra ID` → `Roles & admins` → função → `Assignments` e cruzar com os logs.

- [ ] **Logs de auditoria de alterações administrativas recentes**
  - Entra: `Entra ID` → `Monitoring & health` → `Audit logs`.
  - Filtrar por período, categoria, atividade, iniciador e recurso-alvo.
  - Priorizar: Role management, Application management, User management, Group management, Policy, Conditional Access, Authentication methods e Directory management.
  - Microsoft 365: `Microsoft Purview` → `Audit` → `Search`; filtrar por período, administrador, workload e atividade.

---

## Anexo A — Saídas da CLI

- [ ] **Print do ** `summary.json` ** final — 14/14 success**
  - Dentro da pasta do coletor:

```bash
cat output/summary.json
```

- Para impressão formatada:

```bash
jq . output/summary.json
```

- Validação resumida:

```bash
jq -r '.success, .total, .failed, .results[]? | tostring' output/summary.json
```

- Confirmar que o resultado indica `14/14 success` e preservar o JSON original sem alterações.

- [ ] **Lista dos arquivos gerados na pasta ** `output`**

```bash
find output -maxdepth 1 -type f -printf '%f\n' | sort
```

- Com tamanho e data:

```bash
find output -maxdepth 1 -type f -printf '%TY-%Tm-%Td %TH:%TM  %s bytes  %f\n' | sort
```

- Para salvar a listagem:

```bash
find output -maxdepth 1 -type f -printf '%f\n' | sort > output/file-list.txt
```

### Validação final recomendada

```bash
test -f output/summary.json && echo "summary.json: OK" || echo "summary.json: AUSENTE"
jq empty output/summary.json && echo "JSON válido" || echo "JSON inválido"
find output -maxdepth 1 -type f | sort
```

---

## Referências oficiais

- [Microsoft Entra sign-in logs](https://learn.microsoft.com/en-us/entra/identity/monitoring-health/concept-sign-ins)

- [Microsoft Entra audit logs](https://learn.microsoft.com/en-us/entra/identity/monitoring-health/reference-audit-activities)

- [Configurações de colaboração externa do Microsoft Entra](https://learn.microsoft.com/en-us/entra/external-id/external-collaboration-settings-configure)

- [Privileged Identity Management](https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/pim-configure)
