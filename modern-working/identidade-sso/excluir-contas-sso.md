# Excluir Contas Somente Google do SSO com Entra ID

> **Caminho:** `modern-working/identidade-sso/excluir-contas-sso.md`  
> **Objetivo:** Isolar contas que existem apenas no Google Workspace em uma Unidade Organizacional (OU) sem perfil SAML, permitindo login direto com senha do Google sem erro `AADSTS50034` / `AADSTS50105`.

---

## 1. Pré-requisitos

- Acesso **Super Admin** no Google Admin (`admin.google.com`).
- Relação das caixas departamentais ou técnicas que existem apenas no Google (e não no Entra ID).

---

## 2. Passo a passo

### 2.1 Criar OU Dedicada e Mover as Contas Locais

1. No Google Admin, acesse **Diretório** > **Unidades organizacionais** e crie a OU `Contas Google - Sem SSO`.
2. Vá em **Diretório** > **Usuários**, selecione as contas que não existem no Entra ID, clique em **Mais opções** > **Alterar unidade organizacional** e mova-as para `Contas Google - Sem SSO`.

### 2.2 Desativar o Perfil SSO na OU Criada

1. Acesse **Segurança** > **Autenticação** > **SSO com IdP de terceiros** > **Gerenciar atribuições de perfil de SSO**.
2. No painel esquerdo, selecione a OU `Contas Google - Sem SSO`.
3. Em **Selecionar perfil de SSO**, marque **Nenhum** e clique em **Substituir** (*Override*) / **Salvar**.

---

## 3. Validação e Pontos de Atenção

1. Confirme se o status ao lado da OU aparece como **Substituído**.
2. Em janela anônima (`https://accounts.google.com`), teste uma conta da OU `Contas Google - Sem SSO`: o Google deve solicitar a senha local imediatamente, sem redirecionar para a Microsoft.
