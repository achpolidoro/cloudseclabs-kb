# Implantação de Backup Acronis no Google Workspace

> **Caminho:** `modern-working/google-workspace/backup-acronis-workspace.md`  
> **Objetivo:** Configurar backup cloud-to-cloud do Google Workspace (Gmail, Drive, Contatos e Calendário) utilizando projeto no GCP, Service Account com chave JSON e delegação de domínio (OAuth).

---

## 1. Pré-requisitos

- Acesso de **Super Administrador** no painel Google Workspace (`admin.google.com`).
- Acesso ao console **Google Cloud Platform** (`console.cloud.google.com`) com a mesma conta administrativa.
- Acesso ao painel **Acronis Cyber Protect Cloud** (`cloud.acronis.com`).

---

## 2. Passo a passo

### 2.1 Criar Projeto e Habilitar APIs no GCP

1. Acesse `https://console.cloud.google.com` com a conta Super Admin do Workspace.
2. No seletor superior, clique em **Selecionar projeto** > **Novo Projeto** e defina o nome `acronis-backup-gworkspace`.
3. Com o projeto selecionado, acesse **APIs e Serviços** > **Biblioteca** e ative as 4 APIs abaixo:

| API no GCP | Finalidade no Backup |
| :--- | :--- |
| **Gmail API** | Backup e restauração de caixas de correio |
| **Google Drive API** | Backup e restauração de arquivos e pastas |
| **Google People API** | Backup de contatos |
| **Google Calendar API** | Backup de agendas e eventos |

### 2.2 Criar Service Account e Chave JSON

1. No menu do GCP, acesse **IAM e Administrador** > **Contas de serviço** > **+ Criar Conta de Serviço**.
2. Nomeie como `acronis-backup-sa` e conclua sem atribuir papéis adicionais de projeto.
3. Abra a conta criada, acesse a aba **Chaves** > **Adicionar chave** > **Criar nova chave** (formato **JSON**).
4. Guarde o arquivo `.json` baixado em cofre de senhas seguro.
5. Na aba **Detalhes** da Service Account, copie o número do **ID exclusivo (Client ID)**.

### 2.3 Configurar Tela de Permissão OAuth e Delegação no Workspace

1. No GCP, acesse **APIs e Serviços** > **Tela de permissão OAuth**, selecione o tipo **Interno**, defina o nome `Acronis Backup` e salve.
2. Acesse `https://admin.google.com` > **Segurança** > **Acesso e controles de dados** > **Controles de API**.
3. Clique em **Gerenciar delegação de toda a empresa de domínio** > **Adicionar novo**.
4. No campo **ID do cliente**, cole o ID exclusivo da Service Account.
5. No campo **Escopos OAuth**, cole exatamente a lista abaixo e clique em **Autorizar**:

```text
https://mail.google.com/,https://www.googleapis.com/auth/drive,https://www.googleapis.com/auth/contacts,https://www.googleapis.com/auth/calendar
```

### 2.4 Vincular Organização no Console Acronis

1. No painel Acronis, acesse **Backup** > **Google Workspace** > **Adicionar organização Google Workspace**.
2. Informe o e-mail de um Super Admin ativo e envie o arquivo `.json` da Service Account.
3. Crie o **Plano de Proteção** (diário, cobrindo Gmail, Drive, Contatos e Calendário) e aplique aos usuários.

---

## 3. Validação e Pontos de Atenção

1. Aguarde de 10 a 15 minutos após autorizar os escopos OAuth antes de testar a conexão no Acronis.
2. Execute o primeiro backup manualmente e acompanhe em **Monitoramento** > **Atividades**.
3. Realize obrigatoriamente um teste de restauração de e-mail, arquivo do Drive e contato para validar a integridade do backup.
