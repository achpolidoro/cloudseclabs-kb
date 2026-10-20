# SSO SAML e Provisionamento: Entra ID como IdP para Google

> **Caminho:** `modern-working/identidade-sso/sso-saml-entra-google.md`  
> **Objetivo:** Configurar autenticação única (SSO via SAML) e provisionamento automático de usuários tendo o Microsoft Entra ID como provedor de identidade (IdP) e o Google Workspace / Cloud Identity como provedor de serviço (SP).

---

## 1. Pré-requisitos

- Domínio verificado tanto no **Google Admin** quanto no **Microsoft Entra ID**.
- Conta **Super Admin** no Google Admin (não sofre redirecionamento SAML, evitando bloqueio acidental).
- Conta **Administrador Global** ou **Administrador de Aplicativos** no Entra ID.

---

## 2. Passo a passo

### 2.1 Adicionar Conector e Configuração SAML Inicial no Entra ID

1. No Entra ID (`https://entra.microsoft.com`), acesse **Aplicativos empresariais** > **+ Novo aplicativo**.
2. Busque e adicione **Google Cloud / G Suite Connector by Microsoft**.
3. Em **Usuários e grupos**, atribua o usuário ou grupo de teste.
4. Em **Single sign-on** > **SAML** > **1. Basic SAML Configuration**, preencha os valores temporários para liberar o download do certificado:

```text
Identifier (Entity ID): google.com/a/<seudominio.com.br>
Reply URL (ACS):        https://www.google.com/acs
Sign on URL:            https://www.google.com/a/<seudominio.com.br>/ServiceLogin?continue=https://console.cloud.google.com
```

5. Na seção **3. SAML Certificates**, baixe o **Certificate (Base64)** (`.cer`) e copie o **Login URL** e o **Microsoft Entra Identifier**.

### 2.2 Criar Perfil SAML no Google Admin

1. Acesse `https://admin.google.com/ac/security/sso` > **Add SAML profile**.
2. Defina o nome `Entra ID - SAML`, marque o envio de *login_hint* e preencha:

```text
IdP entity ID:     https://sts.windows.net/<tenant-id>/
Sign-in page URL:  https://login.microsoftonline.com/<tenant-id>/saml2
Sign-out page URL: https://login.microsoftonline.com/common/wsfederation?wa=wsignout1.0
```

3. Envie o certificado `.cer` baixado do Entra ID e salve.
4. Copie os dois valores gerados pelo Google em **Service provider (SP) details**:
   - `Entity ID`: `https://accounts.google.com/samlrp/<rpid>`
   - `ACS URL`: `https://accounts.google.com/samlrp/<rpid>/acs`

### 2.3 Atualizar URLs Finais no Entra ID e Ativar no Google

1. Volte ao Entra ID > **Single sign-on** > **Basic SAML Configuration** e substitua o **Identifier** e a **Reply URL** pelos valores com `<rpid>` gerados pelo Google (marcando como *Default* e removendo os antigos).
2. No Google Admin (`Segurança > Autenticação > SSO com IdP de terceiros`), clique em **Gerenciar atribuições de perfil de SSO** e atribua o perfil `Entra ID - SAML` à OU desejada.
3. Em **Segurança > Autenticação > Desafios de login**, abra **Post-SSO verification** e desative desafios extras pós-SSO para evitar bloqueio após o retorno do SAML.

### 2.4 Provisionamento Automático (Opcional)

1. No aplicativo do Entra ID, acesse **Provisioning** > **Automatic** > **Authorize**.
2. Autentique com a conta Super Admin do Google, clique em **Test Connection**, salve os mapeamentos e mude o **Provisioning Status** para **On**.

---

## 3. Validação e Pontos de Atenção

1. Abra uma janela anônima em `https://accounts.google.com` e informe o e-mail de teste.
2. O navegador deve redirecionar para a tela de login da Microsoft e, após autenticar, abrir a sessão no Google.
3. Todo usuário provisionado pelo Entra ID precisa ter os campos **Nome** (*GivenName*) e **Sobrenome** (*Surname*) preenchidos; caso contrário, a API do Google rejeita a criação.
