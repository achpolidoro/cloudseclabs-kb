# ☁ Acronis Cloud Backup — Google Workspace

## Guia de Implantação em Laboratório e Produção

> [!info] Sobre este documento Guia técnico completo para implantação do Acronis Cyber Protect Cloud com Google Workspace integrado ao Google Workspace. Cobre as 4 fases do projeto: GCP → Workspace Admin → Acronis → Operação.

---

## 📋 Índice

- [[#⚙ Pré-Requisitos]]
- [[#FASE 1 — Google Cloud Platform (GCP)]]
    - [[#1.1 — Acessar o Google Cloud Platform]]
    - [[#1.2 — Criar um Novo Projeto]]
    - [[#1.3 — Habilitar as APIs Necessárias]]
    - [[#1.4 — Criar a Service Account]]
    - [[#1.5 — Gerar a Chave JSON]]
    - [[#1.6 — Configurar o OAuth Consent Screen]]
    - [[#1.7 — Delegar Autoridade no Workspace Admin]]
- [[#FASE 2 — Google Workspace Admin]]
    - [[#2.1 — Criar a Conta Técnica]]
    - [[#2.2 — Verificar os Usuários]]
- [[#FASE 3 — Acronis Cyber Protect Cloud]]
    - [[#3.1 — Criar o Tenant do Cliente]]
    - [[#3.2 — Ativar o Serviço Google Workspace Backup]]
    - [[#3.3 — Adicionar a Organização Google Workspace]]
    - [[#3.4 — Criar o Plano de Proteção]]
    - [[#3.5 — Aplicar o Plano aos Usuários]]
    - [[#3.6 — Configurar Alertas e Notificações]]
- [[#FASE 4 — Operação]]
    - [[#4.1 — Executar o Backup Inicial]]
    - [[#4.2 — Monitoramento dos 7 Dias]]
    - [[#4.3 — Testes de Restore]]
    - [[#4.4 — Passagem de Conhecimento]]
    - [[#4.5 — Checklist Final de Entrega]]
- [[#📚 Documentação Oficial]]
- [[#⚠ Erros Comuns e Soluções]]

---

## ⚙ Pré-Requisitos

> [!warning] Antes de começar Confirme que você possui **todos** os itens abaixo antes de iniciar qualquer fase.

- [ ] Acesso de **Admin Global** ao painel Acronis (conta parceiro)
- [ ] Contato com o responsável de TI do cliente Organização
- [ ] Sessão de screenshare agendada com o cliente (mín. 60 min)
- [ ] Computador com Chrome atualizado
- [ ] E-mail do Super Admin atual do Google Workspace do cliente

---

## FASE 1 — Google Cloud Platform (GCP)

> [!info] Por que o GCP é necessário? O Acronis **não usa OAuth simples**. Ele exige um Google Cloud Project próprio com APIs habilitadas e uma Service Account com chave JSON para autenticar de forma segura e permanente.

### 1.1 — Acessar o Google Cloud Platform

```
URL: https://console.cloud.google.com
```

> [!warning] Quem deve fazer login A conta que fizer login no GCP **precisa ser Super Administrador** do Google Workspace do cliente (`admin@seudominio.com.br`). Execute via screenshare ou use a conta técnica criada na Fase 2.

---

### 1.2 — Criar um Novo Projeto

1. No canto superior esquerdo, clique em **"Selecionar projeto"**
2. Na janela que abrir, clique em **"Novo Projeto"** (canto superior direito)
3. Preencha os campos:

|Campo|Valor sugerido|
|---|---|
|Nome do projeto|`acronis-backup-gworkspace`|
|Organização|Selecionar domínio do cliente (se aparecer)|
|Local|Padrão ou organização do cliente|

4. Clique em **"Criar"**
5. Aguarde a notificação de criação (ícone de sino)
6. Selecione o projeto recém-criado para ativá-lo

> [!check] Confirme O nome do projeto deve aparecer no topo da tela antes de continuar.

---

### 1.3 — Habilitar as APIs Necessárias

O Acronis precisa de **4 APIs** habilitadas:

|API|Para que serve|
|---|---|
|Gmail API|Backup dos e-mails|
|Google Drive API|Backup dos arquivos do Drive|
|Google People API|Backup dos Contatos|
|Calendar API|Backup dos Calendários|

**Como habilitar cada uma:**

1. Menu lateral → **"APIs e Serviços"** → **"Biblioteca"**
2. Pesquise pelo nome da API na barra de busca
3. Clique no resultado correto
4. Clique no botão azul **"Ativar"**
5. Repita para as 4 APIs

> [!check] Como verificar Vá em **"APIs e Serviços" → "APIs e Serviços ativados"** — as 4 APIs devem aparecer na lista.

**Checklist de APIs:**

- [ ] Gmail API — ativada
- [ ] Google Drive API — ativada
- [ ] Google People API — ativada
- [ ] Calendar API — ativada

---

### 1.4 — Criar a Service Account

> [!info] O que é uma Service Account? É uma conta técnica de aplicação (não de usuário) que o Acronis usará para se autenticar no Google de forma autônoma.

1. Menu lateral → **"IAM e Administrador"** → **"Contas de serviço"**
2. Clique em **"+ Criar Conta de Serviço"**
3. Preencha:

|Campo|Valor|
|---|---|
|Nome da conta|`acronis-backup-gworkspace`|
|ID da conta|(preenchido automaticamente — não alterar)|
|Descrição|`Service account para backup Acronis Google Workspace`|

4. Clique em **"Criar e Continuar"**
5. Etapa **"Conceder acesso ao projeto"** → **não preencher** → clique em **"Continuar"**
6. Etapa **"Conceder acesso aos usuários"** → **não preencher** → clique em **"Concluído"**

> [!check] A Service Account deve aparecer na lista da página.

---

### 1.5 — Gerar a Chave JSON

> [!danger] Etapa crítica O arquivo JSON gerado é equivalente a uma **senha de acesso a todos os dados do cliente**. Nunca compartilhe por e-mail ou chat. Guarde em local seguro e criptografado.

1. Na lista de Service Accounts, **clique na conta** `acronis-backup-gworkspace`
2. Clique na aba **"Chaves"** (Keys)
3. Clique em **"Adicionar chave"** → **"Criar nova chave"**
4. Selecione o tipo **"JSON"**
5. Clique em **"Criar"**
6. O arquivo JSON será **baixado automaticamente**
7. Renomeie para: `acronis-gworkspace-key.json`

**Localização segura para guardar o arquivo:**

```
📁 Local recomendado: cofre de senhas (Bitwarden, 1Password) 
   ou pasta criptografada dedicada ao projeto
```

---

### 1.6 — Configurar o OAuth Consent Screen

1. Menu lateral → **"APIs e Serviços"** → **"Tela de permissão OAuth"**
2. Em "Tipo de usuário", selecione **"Interno"**
3. Clique em **"Criar"**
4. Preencha:

|Campo|Valor|
|---|---|
|Nome do app|`Acronis Backup`|
|E-mail de suporte|E-mail do admin do cliente|
|E-mail do desenvolvedor|Seu e-mail técnico|

5. Clique em **"Salvar e Continuar"**
6. Nas etapas seguintes (Escopos e Usuários de teste) → clique em **"Salvar e Continuar"** sem alterar nada
7. Clique em **"Voltar ao painel"**

---

### 1.7 — Delegar Autoridade no Workspace Admin

Esta etapa autoriza a Service Account a acessar os dados dos usuários. É feita no **Google Workspace Admin**, não no GCP.

**Passo 1 — Copiar o Client ID da Service Account (no GCP):**

1. Vá em **"IAM e Administrador"** → **"Contas de serviço"**
2. Clique na Service Account `acronis-backup-gworkspace`
3. Na aba **"Detalhes"**, copie o **"ID exclusivo"**

```
Exemplo de ID exclusivo: 118234567890123456789
(número longo — copie exatamente, sem espaços)
```

**Passo 2 — Adicionar delegação no Google Workspace Admin:**

1. Acesse `admin.google.com` com a conta Super Admin do cliente
2. Vá em **"Segurança"** → **"Acesso e controles de dados"** → **"Controles de API"**
3. Role até **"Delegação de toda a empresa de domínio"**
4. Clique em **"Gerenciar delegação de toda a empresa de domínio"**
5. Clique em **"Adicionar novo"**
6. Cole o Client ID copiado no campo **"ID do cliente"**
7. No campo **"Escopos OAuth"**, cole:

```
https://mail.google.com/,https://www.googleapis.com/auth/drive,https://www.googleapis.com/auth/contacts,https://www.googleapis.com/auth/calendar
```

8. Clique em **"Autorizar"**
9. Confirme que a Service Account aparece na lista com os escopos

> [!info] Propagação Após esta etapa, aguarde **10 a 15 minutos** para a delegação propagar no Google antes de tentar conectar no Acronis.

---

## FASE 2 — Google Workspace Admin

### 2.1 — Criar a Conta Técnica

> [!warning] Por que criar esta conta? Contas externas (`@suaempresa.com.br`) **não podem ser Super Admin** no Workspace do cliente. É necessário criar uma conta dentro do domínio do cliente.

1. Acesse `admin.google.com` com as credenciais do cliente
2. Menu lateral → **"Usuários"**
3. Clique em **"+ Adicionar novo usuário"**
4. Preencha:

|Campo|Valor sugerido|
|---|---|
|Nome|`Tecnico`|
|Sobrenome|`Acronis`|
|E-mail|`tecnico.acronis@[domínio do cliente]`|
|Senha|(gerar senha forte — anotar no cofre)|

5. Clique em **"Adicionar novo usuário"**
6. Após criado, clique no usuário → **"Privilégios de administrador"**
7. Ative o toggle de **"Super Admin"**
8. Clique em **"Salvar"**

**Credenciais para guardar:**

```
Login:  tecnico.acronis@[domínio]
Senha:  ________________________
```

> [!warning] Após o projeto Esta conta deve ser **suspensa ou removida** após a conclusão. Incluir no Termo de Aceite.

---

### 2.2 — Verificar os Usuários

1. No Google Workspace Admin → **"Usuários"**
2. Confirme que existem **exatamente 10 usuários ativos**
3. Anote os e-mails dos 10 usuários:

```
1.  ________________________________
2.  ________________________________
3.  ________________________________
4.  ________________________________
5.  ________________________________
6.  ________________________________
7.  ________________________________
8.  ________________________________
9.  ________________________________
10. ________________________________
```

4. Verifique se há usuários **suspensos** (não contam para o backup)

---

## FASE 3 — Acronis Cyber Protect Cloud

### 3.1 — Criar o Tenant do Cliente

```
URL Painel Parceiro: https://cloud.acronis.com
```

1. Login com sua conta de parceiro
2. Menu lateral → **"Clientes"**
3. Clique em **"+ Novo cliente"**
4. Preencha:

|Campo|Valor|
|---|---|
|Nome|`Organização`|
|E-mail de contato|E-mail do responsável do cliente|
|Modo de cobrança|Conforme contrato|

5. Clique em **"Criar"**
6. Guarde o link de acesso ao tenant gerado

**Dados do Tenant:**

```
URL do painel:  ________________________________
Login:          ________________________________
Senha inicial:  ________________________________
```

---

### 3.2 — Ativar o Serviço Google Workspace Backup

1. Dentro do tenant Organização → **"Serviços"** → **"Gerenciar serviços"**
2. Localize **"Google Workspace Backup"** (pode aparecer como "Cloud-to-Cloud Backup")
3. Clique em **"Ativar"**
4. Selecione o datacenter:

> [!danger] Escolha permanente Para clientes brasileiros: **Brasil** (se disponível) ou **EUA Leste**. Esta escolha é **PERMANENTE** — não pode ser alterada depois sem perder todos os backups.

5. Confirme a ativação

---

### 3.3 — Adicionar a Organização Google Workspace

1. No painel do tenant → **"Backup"** → **"Google Workspace"**
2. Clique em **"Adicionar organização Google Workspace"**
3. Informe o e-mail do Super Admin:

```
tecnico.acronis@[domínio do cliente]
```

4. Clique em **"Procurar"** e selecione o arquivo `acronis-gworkspace-key.json`
    - Ou cole o conteúdo JSON diretamente no campo de texto
5. Clique em **"Confirmar"**

**O que acontece após confirmar:**

- Validação das credenciais com o Google
- Sincronização da lista de usuários (2–5 min)
- Os 10 usuários aparecem na aba "Usuários"

> [!bug] Se aparecer erro de autenticação
> 
> 1. Verifique se os escopos OAuth foram adicionados corretamente (Passo 1.7)
> 2. Aguarde mais 10–15 min para propagação do Google
> 3. Confirme que o arquivo JSON é da Service Account correta

---

### 3.4 — Criar o Plano de Proteção

1. **"Backup"** → **"Planos de proteção"**
2. Clique em **"Criar plano"**
3. Selecione **"Google Workspace"** como tipo
4. Configure:

|Configuração|Valor recomendado|
|---|---|
|O que fazer backup|Gmail, Drive, Contatos, Calendário|
|Frequência|Diário|
|Horário|01:00 AM|
|Fuso horário|America/Sao_Paulo|
|Retenção diária|30 dias|
|Retenção semanal|6 meses|
|Retenção mensal|1 ano|

5. Nome do plano: `Backup-GWorkspace-Diario`
6. Clique em **"Salvar"**

---

### 3.5 — Aplicar o Plano aos Usuários

1. **"Backup"** → **"Google Workspace"** → aba **"Usuários"**
2. Selecione **todos os 10 usuários** (checkbox no topo)
3. Clique em **"Aplicar plano de proteção"**
4. Selecione `Backup-GWorkspace-Diario`
5. Confirme

> [!check] Status esperado Cada usuário deve mostrar **"Protegido"** ou **"Aguardando primeiro backup"**.

---

### 3.6 — Configurar Alertas e Notificações

1. **"Configurações"** → **"Notificações"**
2. Configure alertas por e-mail:

|Evento|Envio|Destinatário|
|---|---|---|
|Backup falhou|Imediato|Você + responsável cliente|
|Concluído com avisos|Imediato|Você + responsável cliente|
|Relatório diário|08:00|Responsável cliente|

3. Clique em **"Enviar e-mail de teste"** para confirmar

---

## FASE 4 — Operação

### 4.1 — Executar o Backup Inicial

> [!info] Primeiro backup = backup completo Copia **tudo do zero**. Pode levar horas dependendo do volume. Os seguintes serão incrementais.

1. Na aba **"Usuários"**, selecione todos os 10
2. Clique em **"Executar agora"**
3. Confirme a execução
4. Acompanhe em **"Atividades"** → **"Monitoramento"**

**Erros comuns:**

|Erro|Causa|Solução|
|---|---|---|
|`Insufficient permissions`|Escopo OAuth incompleto|Revisar delegação (Passo 1.7)|
|`API rate limit exceeded`|Limite do Google atingido|Aguardar 1–2h. Tentativa automática.|
|`User not found`|Usuário desativado/removido|Confirmar no Workspace Admin|
|`Invalid credentials`|JSON incorreto ou expirado|Regerar a chave JSON no GCP|

**Registro do backup inicial:**

|Usuário|Tamanho (GB)|Data/Hora|Status|
|---|---|---|---|
|||||
|||||
|||||
|||||
|||||
|||||
|||||
|||||
|||||
|||||

---

### 4.2 — Monitoramento dos 7 Dias

**Checklist diário (toda manhã — ~10 min):**

- [ ] Acessar Acronis → Monitoramento → Atividades
- [ ] Verificar status dos jobs da noite anterior
- [ ] Confirmar que todos os 10 usuários fizeram backup
- [ ] Verificar aba "Alertas" por qualquer aviso
- [ ] Anotar falhas com hora, usuário e mensagem de erro

**Tabela de severidade:**

|Situação|Severidade|Ação|
|---|---|---|
|1–2 usuários falharam|🟡 Baixa|Investigar e corrigir no dia|
|3+ usuários falharam|🟠 Média|Corrigir e notificar cliente|
|Todos os usuários falharam|🔴 Alta|Verificar OAuth/JSON imediatamente|
|Backup rodando há +12h|🟠 Média|Cancelar e reiniciar o job|

**Registro dos 7 dias:**

|Dia|Data|Sucesso|Falha|Observações|
|---|---|---|---|---|
|1|||||
|2|||||
|3|||||
|4|||||
|5|||||
|6|||||
|7|||||

---

### 4.3 — Testes de Restore

> [!danger] Obrigatório Backup sem teste de restore comprovado **não é backup**. Execute os 3 testes e documente com prints.

#### Teste 1 — Restore de E-mails (Gmail)

1. **"Backup"** → **"Google Workspace"** → selecionar usuário de teste
2. Aba **"Gmail"** → selecionar ponto de restauração (dia anterior)
3. Selecionar **5 e-mails** quaisquer
4. **"Recuperar"** → **"Recuperar para o local original"**
5. Acessar o Gmail do usuário e confirmar presença dos e-mails

```
Resultado: [ ] Sucesso  [ ] Falha
Data do teste: ____________
Evidência: (print anexado)
```

#### Teste 2 — Restore de Arquivo do Drive

1. Aba **"Google Drive"** → navegar até pasta não crítica (ex: "Downloads")
2. Selecionar **1 arquivo** qualquer
3. **"Recuperar"** → **"Recuperar para o local original"**
4. Confirmar no Google Drive que o arquivo está acessível e abrível

```
Resultado: [ ] Sucesso  [ ] Falha
Data do teste: ____________
Evidência: (print anexado)
```

#### Teste 3 — Restore de Contatos

1. Aba **"Contatos"** → selecionar todos os contatos de 1 usuário
2. **"Recuperar"** → **"Recuperar para o local original"**
3. Confirmar nos Contatos do Google do usuário

```
Resultado: [ ] Sucesso  [ ] Falha
Data do teste: ____________
Evidência: (print anexado)
```

---

### 4.4 — Passagem de Conhecimento

**Agenda da reunião (90 minutos):**

|Tempo|Tópico|
|---|---|
|00:00–00:15|Visão geral do que foi implantado e por quê|
|00:15–00:30|Tour pelo painel Acronis — onde fica cada função|
|00:30–00:45|Como verificar o status do backup diariamente|
|00:45–01:00|Como executar um restore (demonstração ao vivo)|
|01:00–01:10|Como interpretar alertas e escalar problemas|
|01:10–01:25|Perguntas, dúvidas e entrega das credenciais|
|01:25–01:30|Assinatura do Termo de Aceite|

**O cliente deve saber fazer sozinho ao final:**

- [ ] Acessar o painel Acronis
- [ ] Verificar status dos backups do dia anterior
- [ ] Identificar usuário com falha no backup
- [ ] Executar um restore simples de e-mails
- [ ] Saber a quem ligar em caso de problema

---

### 4.5 — Checklist Final de Entrega

- [ ] Tenant Acronis criado para Organização
- [ ] Google Cloud Project criado e APIs habilitadas
- [ ] Service Account criada e chave JSON gerada
- [ ] Delegação OAuth configurada no Workspace Admin
- [ ] Organização Google Workspace adicionada no Acronis
- [ ] Plano de proteção criado e aplicado aos 10 usuários
- [ ] Alertas por e-mail configurados
- [ ] Backup inicial executado com sucesso (10/10 usuários)
- [ ] 7 dias de monitoramento realizados (relatório preenchido)
- [ ] Teste de restore — Gmail (5 e-mails)
- [ ] Teste de restore — Google Drive (1 arquivo)
- [ ] Teste de restore — Contatos
- [ ] Passagem de conhecimento realizada
- [ ] Credenciais entregues ao cliente
- [ ] Conta `tecnico.acronis` suspensa ou removida
- [ ] Termo de aceite assinado

---

## 📚 Documentação Oficial

|Recurso|Link|
|---|---|
|Documentação técnica completa|https://www.acronis.com/en/support/documentation/CyberProtectionService/protecting-google-workspace-data.html|
|Knowledge Base — Como adicionar organização GW|https://kb.acronis.com/content/70942|
|Blog com tutorial detalhado|https://www.acronis.com/en/blog/posts/how-to-backup-google-workspace/|
|Integração técnica com Google Workspace|https://solutions.acronis.com/en-us/integrations/google-workspace/|
|App no Google Workspace Marketplace|https://workspace.google.com/marketplace/app/acronis_scs_cyber_protect_cloud/619399626621|

---

## ⚠ Erros Comuns e Soluções

|Erro|Consequência|Como evitar|
|---|---|---|
|OAuth com conta que não é Super Admin|Integração falha ou incompleta|Confirmar nível de acesso antes|
|Não testar restore|Descobrir backup inválido só na crise|Nunca pular a Fase 4.3|
|Não configurar alertas por e-mail|Não perceber falhas por dias|Fazer antes dos 7 dias de monitoramento|
|Datacenter errado no Acronis|Escolha permanente — perda de dados ao mudar|Confirmar com cliente antes de ativar|
|Não documentar tudo|Sem evidências para o Termo de Aceite|Print em cada etapa concluída|
|Escopos OAuth incompletos|Backup parcial — alguns serviços sem cobertura|Colar todos os 4 escopos de uma vez no Passo 1.7|
|Arquivo JSON perdido|Impossível reautenticar sem regerar|Guardar em cofre de senhas imediatamente após download|

---
