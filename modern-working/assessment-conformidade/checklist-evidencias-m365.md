# Checklist de Verificações e Evidências de Postura M365

> **Caminho:** `modern-working/assessment-conformidade/checklist-evidencias-m365.md`  
> **Objetivo:** Roteiro estruturado de verificação de segurança cobrindo Identidade (Entra ID), Privilégios, Aplicativos, Exchange Online, SharePoint/OneDrive, Teams, Intune e Defender.

---

## 1. Pré-requisitos

- Acesso como **Leitor Global** (*Global Reader*) e **View-Only Recipients** no Exchange Online.
- Saída JSON dos coletores automatizados para cruzamento com os portais administrativos.

---

## 2. Passo a passo

### 2.1. Identidade, MFA e Acesso Condicional (`entra.microsoft.com`)

- **Visão Geral e Licenciamento:** Verificar nome do tenant, domínio primário e licenças ativas (`Entra ID P1/P2`).
- **Security Defaults:** Conferir estado em `Overview > Properties > Manage security defaults`.
- **Acesso Condicional:** Listar políticas em `Protection > Conditional Access > Policies` (avaliar escopo de usuários, aplicativos, condições, controles `Grant/Session` e exclusões).
- **Métodos de Autenticação e MFA:** Em `Protection > Authentication methods > Policies` e `Activity > User registration details`, identificar contas sem MFA forte registrado.
- **Colaboração Externa e Convidados:** Em `External Identities > External collaboration settings`, validar quem pode convidar *Guests* e auditar convidados inativos.

### 2.2. Privilégios e Contas de Emergência (PIM)

- **Global Administrators:** Listar todas as contas com atribuição permanente vs. elegível em `Roles & admins > Global Administrator`.
- **Demais Papéis Críticos:** Auditar atribuições permanentes em *Privileged Role Admin*, *Security Admin*, *Exchange Admin* e *Conditional Access Admin*.
- **Contas Break-Glass:** Confirmar existência de ao menos 2 contas de emergência com autenticação forte FIDO2/chave longa e monitoradas em *Sign-in logs*.

### 2.3. Aplicativos e Consentimento OAuth

- **Consentimento de Usuário:** Em `Enterprise applications > Consent and permissions`, verificar se usuários comuns podem autorizar apps de terceiros sozinhos.
- **Auditoria de Enterprise Apps e App Registrations:** Identificar aplicativos sem *Publisher verified* ou com segredos/certificados expirados ou de validade excessiva.

### 2.4. Segurança de E-mail e Exchange Online (`security.microsoft.com`)

- **Auditoria e Fluxo de E-mail:** Validar `Get-OrganizationConfig | Format-List AuditDisabled` e revisar `Get-TransportRule`.
- **Encaminhamento Externo:** Confirmar bloqueio automático na *Outbound spam filter policy* e listar caixas com `ForwardingSmtpAddress`.
- **Protocolos Legados:** Auditar `PopEnabled`, `ImapEnabled` e `SmtpClientAuthenticationDisabled` via `Get-CASMailbox`.
- **Autenticação de Domínio (SPF, DKIM e DMARC):** Validar registros DNS e assinatura DKIM ativa para todos os domínios verificados.

### 2.5. SharePoint, OneDrive, Teams, Intune e Defender

- **Compartilhamento Externo (SPO/OneDrive):** Em `SharePoint admin center > Policies > Sharing`, revisar nível de compartilhamento e expiração de links anônimos (`Anyone`).
- **Microsoft Teams:** Validar políticas de acesso externo, acesso de convidados e permissão de criação de grupos M365.
- **Intune e Dispositivos:** Revisar `Devices > All devices`, políticas de conformidade e restrições de registro (*Enrollment restrictions*).
- **Microsoft Secure Score:** Registrar pontuação atual e priorizar as recomendações de maior impacto em `Secure score > Improvement actions`.

---

## 3. Validação e Pontos de Atenção

1. Ao analisar logs de entrada com protocolos legados (`Sign-in logs > Client app`), habilite a coluna `Error code`: o código `53003` confirma que a tentativa foi bloqueada com sucesso por Acesso Condicional.
