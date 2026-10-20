# Implementação de Políticas de DLP no Microsoft Purview

> **Caminho:** `modern-working/assessment-conformidade/politicas-dlp-purview.md`  
> **Objetivo:** Estruturar, simular e ativar políticas de Prevenção contra Perda de Dados (DLP) no Microsoft Purview cobrindo Exchange, SharePoint, OneDrive, Teams e Endpoints.

---

## 1. Pré-requisitos

- Licenciamento compatível com **Microsoft Purview Data Loss Prevention** (M365 E3/E5 ou Business Premium/Add-on de conformidade).
- Permissão de **Compliance Administrator** ou **Information Protection Administrator** em `https://purview.microsoft.com`.

---

## 2. Passo a passo

### 2.1 Criar a Política e Definir o Escopo

1. Acesse `https://purview.microsoft.com` > **Prevenção contra Perda de Dados (DLP)** > **Políticas** > **+ Criar política**.
2. Selecione uma categoria de proteção (ex.: **Dados financeiros** ou **Informações de identificação pessoal - LGPD/Brasil**) ou escolha **Personalizado**.
3. Selecione os locais onde a política será monitorada:
   - **Exchange Online** (e-mails enviados para fora da organização)
   - **Sites do SharePoint** e **Contas do OneDrive**
   - **Mensagens de chat e canal do Teams**
   - **Dispositivos (Endpoints)**

### 2.2 Configurar Regras de Detecção e Ações

1. Defina os **Tipos de informações confidenciais** (ex.: CPF, CNPJ, Cartão de Crédito, dados bancários) e a contagem mínima de ocorrências.
2. Configure a condição de disparo (ex.: *O conteúdo é compartilhado fora da organização*).
3. Defina as ações graduais:
   - Exibir **Dicas de Política (Policy Tips)** para educar o usuário no momento da ação.
   - Gerar alerta de severidade para a equipe de segurança.
   - Bloquear o compartilhamento externo (ou exigir justificativa registrada).

### 2.3 Homologação em Modo de Simulação

1. Antes de bloquear em produção, selecione **Executar a política no modo de simulação** com dicas de política habilitadas.
2. Analise os eventos capturados durante 7 a 14 dias em **Explorador de Atividades (Activity Explorer)** para eliminar falsos positivos.
3. Após o ajuste fino, edite a política e altere para **Ativar imediatamente**.

---

## 3. Validação e Pontos de Atenção

1. Realize um teste controlado enviando um documento com dados fictícios de teste para um destinatário externo e valide a exibição da *Policy Tip* e o registro do evento no **Alertas DLP**.
