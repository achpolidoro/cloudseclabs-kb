# Direcionar usuários provisionados pelo Entra ID para uma OU específica no Google

## Sumário

1. Contexto
2. Pré-requisitos
3. Conceitos-chave
4. Passo a passo
5. Mover usuários já provisionados
6. (Opcional) Aplicar o SSO somente na OU do Entra ID
7. Validação
8. Erros comuns

---

## 1. Contexto

Por padrão, os usuários criados no Google pelo provisionamento do Entra ID caem na **organização principal**.

Este POP configura o provisionamento para criar esses usuários direto numa **unidade organizacional (OU) específica**. Isso facilita aplicar políticas (como o SSO) somente aos usuários que vêm do Entra ID.

---

## 2. Pré-requisitos

- [ ] Provisionamento Entra ID → Google já configurado e ativo (app **Google Cloud / G Suite Connector by Microsoft**)
- [ ] Conta **super admin** no Google Admin
- [ ] Conta com permissão de **Administrador Global** ou **Administrador de Aplicativos** no Entra ID

---

## 3. Conceitos-chave

- O conector do Entra ID suporta o atributo de destino **OrgUnitPath**, que define a OU do usuário no Google.
- O valor do OrgUnitPath é o **caminho da OU**, sempre começando com **barra**:
  - OU na raiz: `/<nome-da-ou>`
  - OU dentro de outra: `/<ou-pai>/<nome-da-ou>`
- A OU **precisa existir** no Google antes. O Entra ID não cria a OU.
- **Somente durante a criação do objeto:** a OU é definida só quando o usuário é criado. Movimentações manuais no Google são respeitadas depois.
- **Sempre:** o Entra ID força a OU em todo ciclo e **desfaz** movimentações manuais feitas no Google.
- Recomendação: usar **Somente durante a criação do objeto**.

---

## 4. Passo a passo

### Passo 1 — Criar a OU no Google

**Caminho:** Google Admin > **Diretório** > **Unidades organizacionais**

1. Acesse: `https://admin.google.com`
2. Entre com a conta **super admin**.
3. Vá em **Diretório** > **Unidades organizacionais**.
4. Passe o mouse sobre a organização principal e clique no **+** (**Criar nova unidade organizacional**).
5. **Nome:** `Usuarios Entra ID` (sem acento, para evitar erro no mapeamento).
6. **Unidade organizacional pai:** a organização principal.
7. Clique em **Criar**.

### Passo 2 — Criar o mapeamento OrgUnitPath no Entra ID

**Caminho:** **Entra ID** > **Aplicativos empresariais** > **Google Cloud / G Suite Connector by Microsoft** > **Mapeamento de atributos** > **Provisionar usuários do Microsoft Entra ID**

1. Acesse: `https://entra.microsoft.com`
2. Vá em **Entra ID** > **Aplicativos empresariais**.
3. Abra o app **Google Cloud / G Suite Connector by Microsoft**.
4. No menu da esquerda, clique em **Mapeamento de atributos** (*Attribute mapping*).
5. Clique em **Provisionar usuários do Microsoft Entra ID** (*Provision Microsoft Entra ID Users*).
6. Desça até o fim da lista e clique em **Adicionar novo mapeamento**.
7. Preencha os campos:
   - **Tipo de mapeamento:** `Constante`
   - **Valor constante:** `/Usuarios Entra ID`
   - **Atributo de destino:** `OrgUnitPath`
   - **Aplicar este mapeamento:** `Somente durante a criação do objeto`
8. Clique em **Ok**.
9. Clique em **Salvar**, no topo, e confirme com **Sim**.

> [!warning] Atenção
> O **Valor constante** deve ser **igual** ao nome da OU criada no Passo 1, com a **barra** no início.

---

## 5. Mover usuários já provisionados

O mapeamento vale só para usuários **novos**. Os usuários já provisionados precisam ser movidos manualmente.

**Caminho:** Google Admin > **Diretório** > **Usuários**

1. Vá em **Diretório** > **Usuários**.
2. Marque a caixa dos usuários que vieram do Entra ID.
3. No topo da lista, clique em **Mais opções** (ou nos **3 pontinhos**).
4. Clique em **Alterar unidade organizacional**.
5. Escolha **Usuarios Entra ID**.
6. Clique em **Continuar** e depois em **Alterar**.

---

## 6. (Opcional) Aplicar o SSO somente na OU do Entra ID

Configuração recomendada: o SSO vale **só** para quem vem do Entra ID. Contas criadas direto no Google nunca são redirecionadas por engano.

> [!warning] Antes de fazer
> Confirme que **todos** os usuários do Entra ID já estão na OU **Usuarios Entra ID** (seção 5). Quem ficar fora dela deixa de entrar pelo SSO.

**Caminho:** Google Admin > **Segurança** > **Autenticação** > **SSO com IdP de terceiros** > **Gerenciar atribuições de perfil de SSO**

1. Vá até **Gerenciar atribuições de perfil de SSO** pelo caminho acima.
2. Na esquerda, clique na OU **Usuarios Entra ID**.
3. Em **Selecionar perfil de SSO**, escolha o perfil do Entra ID (ex.: **Entra ID - SAML**).
4. Clique em **Substituir**, se aparecer, e depois em **Salvar**.
5. Na esquerda, clique na **organização principal**.
6. Em **Selecionar perfil de SSO**, escolha **Nenhum**.
7. Clique em **Salvar**.

Resultado:
- OU **Usuarios Entra ID** → login pelo Entra ID
- Demais usuários → login com a senha do Google

---

## 7. Validação

### Mapeamento da OU

1. Crie um usuário de teste no Entra ID, com **Nome** e **Sobrenome** preenchidos e o domínio verificado no Google.
2. Atribua o usuário ao app: **Aplicativos empresariais** > **Google Cloud / G Suite Connector by Microsoft** > **Usuários e grupos** > **+ Adicionar usuário/grupo**.
3. No menu da esquerda do app, clique em **Provisionar sob demanda**, busque o usuário e clique em **Provisionar**.
4. No Google Admin, vá em **Diretório** > **Usuários** e abra o usuário.
5. ✅ **Resultado esperado:** a unidade organizacional do usuário é **Usuarios Entra ID**.

### SSO (se a seção 6 foi aplicada)

1. Em janela anônima, acesse: `https://accounts.google.com`
2. Teste com um usuário da OU **Usuarios Entra ID**. ✅ Deve ser redirecionado para a Microsoft.
3. Teste com uma conta somente Google. ✅ Deve pedir a senha do Google.

---

## 8. Erros comuns

| Sintoma | Causa | Solução |
|---|---|---|
| Falha no provisionamento por OU inválida | O valor do OrgUnitPath não corresponde a uma OU existente | Conferir o nome da OU e a barra no início do valor (Passo 2) |
| O usuário novo cai na organização principal | O mapeamento não foi salvo | Refazer o Passo 2 e clicar em **Salvar** |
| Usuários antigos continuam na organização principal | O mapeamento vale só na criação | Mover manualmente (seção 5) |
| O usuário volta para a OU depois de movido manualmente | Mapeamento configurado como **Sempre** | Alterar para **Somente durante a criação do objeto** |
| Usuário do Entra ID não é mais redirecionado para a Microsoft | Ficou fora da OU **Usuarios Entra ID** após a seção 6 | Mover o usuário para a OU (seção 5) |
