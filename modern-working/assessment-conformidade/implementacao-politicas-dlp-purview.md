# Implementação de Políticas de Prevenção contra Perda de Dados (DLP) no Purview

---

[https://learn.microsoft.com/pt-br/purview/](https://learn.microsoft.com/pt-br/purview/)

## 🧩 Resumo sobre DLP (Data Loss Prevention) – Microsoft Purview

A **Prevenção contra Perda de Dados (DLP)** no Microsoft Purview é uma solução que **ajuda as organizações a identificar, monitorar e proteger informações confidenciais** em toda a sua infraestrutura de dados — incluindo e-mails, documentos, dispositivos, aplicativos em nuvem e endpoints.

Ela permite:

- **Detecção automática** de dados sensíveis (como PII, informações financeiras, segredos corporativos).
- **Criação de políticas** que previnem o compartilhamento não autorizado ou uso indevido de informações.
- **Monitoramento em tempo real** de atividades de dados e geração de alertas.
- **Integração com etiquetas de confidencialidade** e classificadores para proteger dados automaticamente.
- **Controle adaptável** — aplicando diferentes ações com base no contexto (bloquear, alertar, permitir com justificativa).

O DLP do Microsoft Purview funciona em todo o ecossistema Microsoft 365, incluindo:

- **SharePoint, OneDrive e Exchange**
- **Microsoft Teams e Copilot**
- **Aplicações SaaS e navegadores (via Microsoft Edge)**
- **Pontos de extremidade (dispositivos locais)**

Com o DLP, as empresas conseguem **reduzir o risco de vazamento de dados**, **cumprir requisitos regulatórios (GDPR, LGPD, HIPAA)** e **proteger sua propriedade intelectual**.

---

## 🛠️ Tutorial – Como Iniciar, Configurar e Aplicar Políticas de DLP no Microsoft Purview

### 1. ✅ Pré-requisitos

Antes de começar, garanta que:

- Você possui licenças adequadas do Microsoft 365 (E5 ou complementos de conformidade).
- Possui **permissões administrativas** no portal do Microsoft Purview (funções como *Administrador de Proteção de Informações* ou *Administrador de Conformidade*).
- Os serviços que receberão políticas DLP (Exchange, SharePoint, Teams etc.) estejam habilitados.

---

### 2. 🚀 Acessar o Portal do Microsoft Purview

1. Entre no portal: [https://compliance.microsoft.com](https://compliance.microsoft.com/)
2. No menu lateral esquerdo, acesse **“Prevenção contra Perda de Dados”** (*Data Loss Prevention*).
3. Clique em **“Políticas”** para começar a criar sua primeira política DLP.

---

### 3. ⚙️ Criar uma Política DLP

1. **Clique em “+ Criar política”**.
2. Escolha um **modelo pronto** (por exemplo, “Dados financeiros” ou “Informações de identificação pessoal”) ou crie do zero.
3. **Defina o escopo da política**, escolhendo onde ela será aplicada:
    - **Exchange Online** – para proteger e-mails.
    - **SharePoint e OneDrive** – para arquivos e documentos.
    - **Teams** – para conversas e arquivos em chats/canais.
    - **Endpoints** – para dispositivos corporativos.
    - **Aplicações em nuvem / Edge** – para controle de compartilhamento com apps externos.
4. **Configure as regras de detecção:**
    - Tipos de informações confidenciais a serem monitoradas (cartões de crédito, RG, CPF etc.).
    - Condições adicionais (por exemplo, número de ocorrências ou contexto do compartilhamento).
5. **Defina as ações quando a regra for acionada:**
    - Bloquear a ação.
    - Notificar o usuário e registrar um alerta.
    - Permitir com justificativa.
6. **Configure notificações e relatórios:**
    - Ative alertas para administradores e logs de auditoria.
    - Permita que os usuários recebam dicas de política no momento da tentativa de violação.
7. **Revisar e publicar** a política.

---

### 4. 🧪 Testar e Ajustar a Política

- Inicie com o **modo de simulação**, onde as ações não são aplicadas, mas relatórios são gerados.
- Revise os resultados no **Explorador de Atividades** e ajuste as regras conforme necessário.
- Após validar, altere a política para o **modo de aplicação ativa**.

---

### 5. 🔎 Monitoramento e Manutenção

- Use o painel **Explorador de Atividades DLP** para visualizar incidentes, usuários e padrões de violação.
- Gere **relatórios de conformidade** para auditorias e investigações.
- Atualize políticas regularmente conforme surgirem novas regulamentações ou padrões de dados confidenciais.

---

## 🧠 Dicas e Boas Práticas

- Comece com políticas específicas e vá ampliando o escopo gradualmente.
- Utilize **etiquetas de confidencialidade** para automatizar a proteção de arquivos.
- Integre o DLP com o **Microsoft Defender** e **Microsoft Sentinel** para resposta a incidentes.
- Ative a **monitorização em Edge** para impedir vazamentos via web e IA não gerenciada.

---

### ✅ Conclusão

O DLP no Microsoft Purview é uma solução poderosa para **proteger dados sensíveis e atender exigências legais e regulatórias**. Ele centraliza a gestão de políticas, monitora atividades em tempo real e aplica controles em toda a infraestrutura de dados — da nuvem aos dispositivos locais.

---
