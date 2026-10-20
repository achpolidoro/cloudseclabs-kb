# Guia Rápido de Administração Diária no Microsoft 365

> **Caminho:** `modern-working/sharepoint-teams/guia-admin-basica-m365.md`  
> **Objetivo:** Roteiro objetivo de operações administrativas rotineiras em Usuários, Grupos, SharePoint e Microsoft Teams.

---

## 1. Pré-requisitos

- Acesso administrativo com função de menor privilégio adequada (`User Administrator`, `SharePoint Administrator` ou `Teams Administrator`).

---

## 2. Passo a passo

### 2.1 Redefinição de Senha e Encaminhamento (`admin.microsoft.com`)

1. Acesse **Usuários** > **Usuários ativos** e selecione a conta.
2. Para senha: clique em **Redefinir senha** e marque **Exigir que este usuário altere a senha no primeiro acesso**.
3. Para encaminhamento: abra a aba **Email** > **Gerenciar encaminhamento de email**, defina o destinatário interno e marque se deseja manter cópia na caixa original.

### 2.2 Criação de Grupos de Distribuição vs. Grupos M365

1. Acesse **Equipes e grupos** > **Equipes e grupos ativos** > **Adicionar um grupo**.
2. Escolha o tipo conforme a necessidade:
   - **Lista de distribuição:** apenas disparo de e-mail para múltiplos destinatários.
   - **Grupo do Microsoft 365:** cria caixa compartilhada, site no SharePoint e integração com Teams.

### 2.3 Provisionamento de Sites no SharePoint (`admin.microsoft.com/sharepoint`)

1. Acesse **Sites** > **Sites ativos** > **+ Criar**.
2. Escolha entre **Site de equipe** (colaboração com permissões de grupo) ou **Site de comunicação** (intranet/divulgação).
3. Defina sempre ao menos **2 proprietários** para evitar sites órfãos.

### 2.4 Criação e Governança de Equipes no Teams (`admin.teams.microsoft.com`)

1. Acesse **Equipes** > **Gerenciar equipes** > **Adicionar**.
2. Adicione membros e promova ao menos dois responsáveis como **Proprietários**.
3. Em **Políticas de mensagens** e **Acesso externo**, mantenha bloqueado o compartilhamento ou chat com domínios externos não homologados.

---

## 3. Validação e Pontos de Atenção

1. Evite conceder permissão direta a usuários avulsos em pastas do SharePoint; utilize sempre **Grupos de Segurança** ou **Grupos do M365** para facilitar a auditoria.
