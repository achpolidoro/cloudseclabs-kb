# SSO SAML com Entra ID como IdP para Google Cloud Identity e Workspace


## Sumário

1. Contexto
2. Pré-requisitos
3. Conceitos-chave
4. Valores usados neste lab
5. Parte 1 — Criar o Cloud Identity Free (Google)
6. Parte 2 — Preparar o Entra ID (Microsoft)
7. Parte 3 — Criar o perfil SAML (Google)
8. Parte 4 — Finalizar o SAML (Entra ID)
9. Parte 5 — Ativar o SSO e criar o usuário (Google)
10. Validação
11. Erros comuns
12. Próximos passos

---

## 1. Contexto

Este POP configura login único (SSO) via **SAML**: o usuário digita o e-mail no Google, é enviado para a tela da Microsoft, autentica no **Entra ID** e volta logado no Google.

- **Entra ID** = IdP (quem autentica)
- **Google Cloud Identity** = SP (quem recebe o login)

---

## 2. Pré-requisitos

- [ ] Um domínio próprio (ex.: `cloudseclabs.com.br`)
- [ ] Acesso ao DNS do domínio (ex.: Cloudflare)
- [ ] O mesmo domínio **adicionado e verificado** no tenant Microsoft
- [ ] Conta com permissão de **Administrador Global** ou **Administrador de Aplicativos** no Entra
- [ ] Um e-mail pessoal (Gmail) para contato e recuperação da conta Google

> [!info] Licenciamento
> **Entra ID Free** atende: SSO SAML com app de galeria e atribuição de **usuários individuais**.
> Atribuir **grupos** ao app exige **Entra ID P1**.

---

## 3. Conceitos-chave

- **Uma conta Gmail não vira Cloud Identity.** O Google cria uma organização nova, ligada ao domínio. O Gmail serve só como contato.
- **O usuário precisa existir dos dois lados** (Entra e Google) com o **mesmo e-mail**.
- **O super admin do Google não passa pelo SSO.** Ele sempre entra com a senha do Google, então você não fica trancado para fora.
- **Contas novas do Google não têm mais o "Legacy SSO profile".** Usa-se o **Add SAML profile**.
- **O certificado SAML do Entra só pode ser baixado depois que a seção 1 (Basic SAML Configuration) é preenchida.** Por isso entram valores temporários primeiro e os valores finais depois.
- **Cloud Identity Free não expira.** O limite de 14 dias vale só para o trial do Workspace pago.

---

## 4. Valores usados neste lab

| Item | Valor |
|---|---|
| Domínio | `cloudseclabs.com.br` |
| Admin Google | `admin@cloudseclabs.com.br` |
| Usuário de teste | `cloud_identity@cloudseclabs.com.br` |
| Tenant ID (Entra) | `<tenant-id>` |
| Login URL (Entra) | `https://login.microsoftonline.com/<tenant-id>/saml2` |
| Microsoft Entra Identifier | `https://sts.windows.net/<tenant-id>/` |
| Entity ID (Google SP) | `https://accounts.google.com/samlrp/<rpid>` |
| ACS URL (Google SP) | `https://accounts.google.com/samlrp/<rpid>/acs` |

> [!tip] Para reutilizar em outro ambiente
> Troque nos passos: `<dominio>`, `<tenant-id>` e `<rpid>` (o código do final do Entity ID do Google).

---

## 5. Parte 1 — Criar o Cloud Identity Free (Google)

### Passo 1 — Conferir se o domínio já tem organização no Google

1. Abra uma **janela anônima**.
2. Acesse: `https://admin.google.com`
3. Entre com a conta Google que usa o e-mail do domínio (se existir).
4. Veja o resultado:
   - Se **abrir o painel Admin**, a organização já existe. Pule para o **Passo 4**.
   - Se aparecer **"não é conta de administrador"**, é uma conta pessoal. Siga para o **Passo 2**.

### Passo 2 — Criar a conta Cloud Identity Free ⭐

1. Na **janela anônima**, saia de qualquer conta logada.
2. Acesse: `https://workspace.google.com/gcpidentity/signup?sku=identityfree`
3. **Nome da empresa:** nome do lab (ex.: `CloudSecLabs`).
4. **Número de funcionários:** `Só você`.
5. **País:** `Brasil`.
6. **Dados de contato:** seu nome e seu **Gmail pessoal**.
7. **Tem um domínio?** Clique em **Sim** e digite `<dominio>`.
8. **Usuário admin:** crie `admin@<dominio>` com uma senha forte.

> [!warning] Atenção
> O usuário admin **não pode** ter o mesmo e-mail de uma conta Google pessoal que já existe nesse domínio. Se tiver, o Google avisa que o nome já está em uso.

### Passo 3 — Verificar o domínio

1. O Google mostra um **registro TXT**. Copie o valor.
2. No painel de DNS (ex.: Cloudflare), crie um registro:
   - **Tipo:** `TXT`
   - **Nome:** `@`
   - **Conteúdo:** o valor copiado do Google
3. Volte ao Google e clique em **Verificar**.
4. Aguarde a tela **"Your domain is verified!"**.

### Passo 4 — Confirmar que a assinatura é gratuita ⭐

1. Acesse: `https://admin.google.com/ac/billing/subscriptions`
2. Confira se aparece:
   - **Nome:** `Cloud Identity Free`
   - **Status:** `Active`
   - **Payment plan:** `Free plan`
3. **Não clique** em **Buy or upgrade**.

---

## 6. Parte 2 — Preparar o Entra ID (Microsoft)

### Passo 5 — Conferir o domínio no Entra

1. Acesse: `https://entra.microsoft.com/#view/Microsoft_AAD_IAM/DomainsList.ReactView`
2. Confira se `<dominio>` aparece com status **Verified**.
3. Se não aparecer, adicione e verifique o domínio antes de continuar.

### Passo 6 — Criar o usuário de teste

1. Acesse: `https://entra.microsoft.com/#view/Microsoft_AAD_UsersAndTenants/UserManagementMenuBlade/~/AllUsers`
2. Clique em **+ Novo usuário** e depois em **Criar novo usuário**.
3. **Nome principal do usuário:** digite `cloud_identity`.
4. No seletor ao lado, escolha `<dominio>`.
5. **Nome de exibição:** `Cloud Identity`.
6. Deixe a senha **gerada automaticamente** e **copie a senha**.
7. Clique em **Revisar + criar** e depois em **Criar**.

### Passo 7 — Adicionar o app do Google

1. Acesse: `https://entra.microsoft.com/#view/Microsoft_AAD_IAM/AppGalleryBladeV2`
2. Na busca, digite `Google Cloud`.
3. Clique em **Google Cloud / G Suite Connector by Microsoft**.
4. Deixe o nome como está.
5. Clique em **Criar** e aguarde abrir a tela do app.

### Passo 8 — Atribuir o usuário ao app

1. No menu da esquerda do app, clique em **Users and groups**.
2. Clique em **+ Add user/group**.
3. Embaixo de **Users**, clique em **None Selected**.
4. Busque `cloud_identity` e marque o usuário.
5. Clique em **Select**.
6. Clique em **Assign**.

### Passo 9 — Preencher a configuração SAML temporária ⭐

> [!info] Por que temporária?
> O Entra só libera o download do certificado depois que a seção 1 é preenchida. Os valores finais entram no **Passo 13**.

1. No menu da esquerda do app, clique em **Single sign-on**.
2. Escolha **SAML**.
3. Na seção **1 – Basic SAML Configuration**, clique em **Edit**.
4. **Identifier (Entity ID):** clique em **Add identifier** e cole:
```
   google.com/a/<dominio>
```
5. **Reply URL:** clique em **Add reply URL** e cole:
```
   https://www.google.com/acs
```
6. **Sign on URL:** cole:
```
   https://www.google.com/a/<dominio>/ServiceLogin?continue=https://console.cloud.google.com
```
7. Deixe **Relay State** e **Logout Url** em branco.
8. Clique em **Save** e feche o painel no **X**.
9. Se perguntar sobre testar, clique em **No, I'll test later**.

### Passo 10 — Baixar o certificado e copiar os dados do Entra

1. Na seção **3 – SAML Certificates**, em **Certificate (Base64)**, clique em **Download**.
2. Guarde o arquivo **.cer**.
3. Na seção **4 – Set up Google Cloud / G Suite Connector by Microsoft**, copie para um bloco de notas:
   - **Login URL**
   - **Microsoft Entra Identifier**

---

## 7. Parte 3 — Criar o perfil SAML (Google)

### Passo 11 — Criar o perfil SAML ⭐

1. Acesse: `https://admin.google.com/ac/security/sso`
2. Entre com `admin@<dominio>`.
3. Clique em **Add SAML profile**.
4. **SSO profile name:** `Entra ID - SAML`
5. Em **Autofill email**, marque **Send email address in the URL as the *login_hint* parameter**.
6. Em **IDP details**, preencha os campos:
   - **IdP entity ID:**
```
     https://sts.windows.net/<tenant-id>/
```
   - **Sign-in page URL:**
```
     https://login.microsoftonline.com/<tenant-id>/saml2
```
   - **Sign-out page URL:**
```
     https://login.microsoftonline.com/common/wsfederation?wa=wsignout1.0
```
   - **Change password URL:**
```
     https://account.activedirectory.windowsazure.com/changepassword.aspx
```
7. Em **Verification certificate**, clique em **Upload certificate** e escolha o **.cer** baixado no Passo 10.
8. Clique em **Save**.

### Passo 12 — Copiar os dados do Google

1. Depois de salvar, a seção **Service provider (SP) details** mostra dois campos.
2. Copie para o bloco de notas:
   - **Entity ID** (formato `https://accounts.google.com/samlrp/<rpid>`)
   - **ACS URL** (formato `https://accounts.google.com/samlrp/<rpid>/acs`)

---

## 8. Parte 4 — Finalizar o SAML (Entra ID)

### Passo 13 — Trocar os valores temporários pelos definitivos ⭐

1. Volte ao app **Google Cloud / G Suite Connector by Microsoft** e abra **Single sign-on**.
2. Na seção **1 – Basic SAML Configuration**, clique em **Edit**.
3. Em **Identifier (Entity ID)**:
   - Clique em **Add identifier** e cole:
```
     https://accounts.google.com/samlrp/<rpid>
```
   - Marque **Default** na linha nova.
   - Apague a linha `google.com/a/<dominio>` na **lixeira**.
4. Em **Reply URL**:
   - Clique em **Add reply URL** e cole:
```
     https://accounts.google.com/samlrp/<rpid>/acs
```
   - Marque **Default** na linha nova.
   - Apague a linha `https://www.google.com/acs` na **lixeira**.
5. Não altere o **Sign on URL**.
6. Clique em **Save** e feche no **X**.
7. Se perguntar sobre testar, clique em **No, I'll test later**.

---

## 9. Parte 5 — Ativar o SSO e criar o usuário (Google)

### Passo 14 — Atribuir o perfil SAML à organização

1. Acesse: `https://admin.google.com/ac/security/sso`
2. Clique em **Manage SSO profile assignments**.
3. Na esquerda, deixe selecionada a organização principal.
4. Em **Select SSO profile**, escolha **Entra ID - SAML**.
5. Deixe marcado **Have Google prompt for their username, then redirect them to this profile's IDP sign-in page**.
6. Clique em **SAVE**.

### Passo 15 — Criar o usuário no Google

1. Acesse: `https://admin.google.com/ac/users`
2. Clique em **Add new user**.
3. **First name:** `Cloud`
4. **Last name:** `Identity`
5. **Primary email:** digite `cloud_identity` e confira se aparece `@<dominio>`.
6. Deixe a senha **automática**. Ela não é usada, porque o login é feito pelo Entra.
7. Clique em **Add new user**.

### Passo 16 — Desligar a verificação pós-SSO ⭐

> [!warning] Por que fazer?
> Sem isso, depois do login na Microsoft o Google pode mostrar **"verificar se é você"** e travar nessa tela.

1. Acesse: `https://admin.google.com`
2. No menu da esquerda, clique em **Security**, depois em **Authentication**, depois em **Login challenges**.
3. Abra a seção **Post-SSO verification**.
4. **Desmarque** a opção dos usuários que entram via **third-party IdP**.
5. Clique em **Save**.
6. Aguarde **5 minutos**.

---

## 10. Validação

1. Feche todas as janelas anônimas.
2. Abra uma **janela anônima nova**.
3. Acesse: `https://myaccount.google.com`
4. Digite `cloud_identity@<dominio>` e clique em **Avançar**.
5. ✅ O Google deve **redirecionar para a tela da Microsoft**.
6. Entre com a senha do Entra.
7. No primeiro acesso, o Entra pede para **trocar a senha**. Troque.
8. Se o Google mostrar boas-vindas ou termos, clique em **Aceitar**.
9. ✅ **Resultado esperado:** a conta Google abre logada como `cloud_identity`.

**Conferência extra no Entra:**
- App > **Sign-in logs**: o login do `cloud_identity` deve aparecer como **Success**.

---

## 11. Erros comuns

| Sintoma | Causa | Solução |
|---|---|---|
| `admin.google.com` diz que não é admin | A conta do domínio é pessoal (não gerenciada) | Criar o Cloud Identity Free (Passo 2) |
| Google diz que o nome do admin já está em uso | Já existe uma conta pessoal com esse e-mail | Usar outro nome, ex.: `admin@` |
| Download do certificado está cinza | Seção 1 do SAML vazia | Preencher os valores temporários (Passo 9) |
| Não existe "Legacy SSO profile" | Contas novas do Google não têm mais esse perfil | Usar **Add SAML profile** (Passo 11) |
| Google pede a senha dele em vez de ir pra Microsoft | A atribuição do perfil ainda não propagou | Aguardar 5–10 minutos e testar de novo |
| Trava em "Google precisa verificar se é você" | Verificação pós-SSO ativa | Desligar em **Login challenges** (Passo 16) |
| Erro de usuário não encontrado no Google | Usuário não existe no Google | Criar com o mesmo e-mail do Entra (Passo 15) |
| Erro de acesso negado no Entra | Usuário não está atribuído ao app | Atribuir em **Users and groups** (Passo 8) |

> [!warning] Conta pessoal existente no domínio
> Não crie no Google (nem provisione pelo Entra) um usuário com o **mesmo e-mail** de uma conta Google pessoal já existente no domínio, para não gerar conflito de conta.
> Neste lab: `usuario.pessoal@seudominio.com.br`.

---

## 12. Próximos passos

- [ ] Confirmar o login após desligar a verificação pós-SSO
- [ ] (Opcional) Configurar o **provisionamento automático** no app do Entra (menu **Provisioning**), para não precisar criar usuários manualmente no Google
- [ ] (Opcional) Transferir a conta pessoal antiga para a organização em `https://admin.google.com/ac/list/unmanagedusers`
- [ ] Remover o usuário de teste ao final do lab
