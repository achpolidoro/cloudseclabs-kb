# Excluir contas somente Google do SSO com Entra ID

## Sumário

1. Contexto
2. Sintoma
3. Causa
4. Pré-requisitos
5. Conceitos-chave
6. Passo a passo
7. Validação
8. Erros comuns
9. Observações

---

## 1. Contexto

Depois da ativação do SSO SAML (Entra ID como IdP) no Google Workspace, algumas contas que existem **somente no Google** pararam de conseguir fazer login. Exemplos: contas de setor como `rh@<dominio>` e `ti@<dominio>`.

Este POP separa essas contas numa unidade organizacional (OU) própria, sem SSO, para que voltem a entrar com a **senha do Google**.

---

## 2. Sintoma

- O usuário digita o e-mail no Google e é **redirecionado para a tela da Microsoft**.
- Na Microsoft, o login falha, porque a conta **não existe no Entra ID** ou **não está atribuída ao app**.
- Erros comuns na tela da Microsoft:
  - **AADSTS50034:** o usuário não existe no Entra ID.
  - **AADSTS50105:** o usuário existe, mas não está atribuído ao app.

---

## 3. Causa

O perfil SSO foi atribuído à **organização principal**. Com isso, **todos os usuários** do Google são enviados ao Entra ID, inclusive os que não existem lá.

---

## 4. Pré-requisitos

- [ ] Conta **super admin** no Google Admin
- [ ] Lista das contas que existem **somente no Google** (não estão no Entra ID)

---

## 5. Conceitos-chave

- A atribuição do perfil SSO é feita **por OU ou por grupo**.
- Uma OU filha **herda** as configurações da OU pai, a menos que a configuração seja **substituída** nela.
- Ao definir o perfil SSO como **Nenhum** numa OU, os usuários dessa OU entram com a **senha do Google**.
- A OU nova herda **todas as outras políticas** da organização principal (apps, segurança etc.). Só muda o que for configurado nela.
- **Super admins** não passam pelo SSO, independentemente da OU.

---

## 6. Passo a passo

### Passo 1 — Criar a OU das contas somente Google

**Caminho:** Google Admin > **Diretório** > **Unidades organizacionais**

1. Acesse: `https://admin.google.com`
2. Entre com a conta **super admin**.
3. Vá em **Diretório** > **Unidades organizacionais**.
4. Passe o mouse sobre a organização principal e clique no **+** (**Criar nova unidade organizacional**).
5. **Nome:** `Contas Google - Sem SSO`
6. **Unidade organizacional pai:** a organização principal.
7. Clique em **Criar**.

### Passo 2 — Mover as contas para a OU

**Caminho:** Google Admin > **Diretório** > **Usuários**

1. Vá em **Diretório** > **Usuários**.
2. Marque a caixa de cada conta que existe **somente no Google** (ex.: `rh@<dominio>`, `ti@<dominio>`).
3. No topo da lista, clique em **Mais opções** (ou nos **3 pontinhos**).
4. Clique em **Alterar unidade organizacional**.
5. Escolha **Contas Google - Sem SSO**.
6. Clique em **Continuar** e depois em **Alterar**.

### Passo 3 — Remover o SSO da OU

**Caminho:** Google Admin > **Segurança** > **Autenticação** > **SSO com IdP de terceiros** > **Gerenciar atribuições de perfil de SSO**

1. Vá até **Gerenciar atribuições de perfil de SSO** pelo caminho acima.
2. Na esquerda, clique na OU **Contas Google - Sem SSO**.
3. Em **Selecionar perfil de SSO**, escolha **Nenhum**.
4. Clique em **Substituir**, se aparecer, e depois em **Salvar**.
5. Confira se, ao lado da OU, aparece **Substituído**. Isso indica que ela não herda mais o SSO da organização principal.

---

## 7. Validação

1. Aguarde alguns minutos após salvar.
2. Peça para uma das contas movidas abrir uma **janela anônima**.
3. Acesse: `https://accounts.google.com`
4. Digite o e-mail da conta (ex.: `rh@<dominio>`) e clique em **Avançar**.
5. ✅ **Resultado esperado:** o Google pede a **senha do Google**, sem redirecionar para a Microsoft.
6. Teste também com um usuário que **existe no Entra ID**. Ele deve continuar sendo **redirecionado para a Microsoft**.

---

## 8. Erros comuns

| Sintoma | Causa | Solução |
|---|---|---|
| A conta continua indo para a Microsoft | A mudança ainda não propagou | Aguardar alguns minutos e testar de novo em janela anônima |
| A conta continua indo para a Microsoft | A OU não ficou como **Substituído** | Refazer o Passo 3 e clicar em **Substituir** antes de salvar |
| A conta continua indo para a Microsoft | O usuário não foi movido para a OU | Conferir a OU do usuário em **Diretório** > **Usuários** |
| A conta não entra nem com a senha do Google | Senha desconhecida ou conta suspensa | Conferir o status em **Diretório** > **Usuários** e redefinir a senha, se necessário |

---

## 9. Observações

- **Contas novas somente Google:** devem ser criadas (ou movidas) direto na OU **Contas Google - Sem SSO**.
- **Alternativa:** se a conta precisar entrar pelo Entra ID, criar o usuário no Entra ID com o **mesmo e-mail** e atribuir ao app **Google Cloud / G Suite Connector by Microsoft** (**Usuários e grupos**). Nesse caso ela fica fora desta OU.
- **Boa prática para novas implantações:** atribuir o SSO primeiro a uma **OU ou grupo piloto**, e não à organização principal, para evitar este cenário.
