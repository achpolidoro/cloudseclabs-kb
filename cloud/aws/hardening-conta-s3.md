# Hardening Inicial de Conta AWS e Bloqueio Público S3

> **Caminho:** `cloud/aws/hardening-conta-s3.md`  
> **Objetivo:** Proteger a conta raiz da AWS, configurar acesso administrativo via IAM Identity Center e bloquear exposição acidental de buckets Amazon S3 no nível da conta.

## 1. Pré-requisitos
- Acesso inicial à conta raiz (*Root User*) da AWS recém-criada.
- Dispositivo gerador de MFA compatível com FIDO2/WebAuthn ou TOTP.
- AWS CLI instalada e autenticada para validação via linha de comando.

## 2. Passo a passo

### 2.1. Proteção da Conta Raiz e IAM Identity Center
1. Acesse o console AWS com a conta raiz, abra **IAM** > **Security recommendations** e habilite **MFA** imediatamente no usuário raiz.
2. Remova qualquer par de chaves de acesso (*Access Keys*) associado à conta raiz.
3. Ative o **AWS IAM Identity Center** e crie usuários/grupos administrativos com permissão delegada, encerrando o uso cotidiano do usuário raiz.

### 2.2. Bloqueio de Acesso Público no S3 (Nível da Conta)
1. Acesse o console do **Amazon S3** > **Block Public Access (account settings)** no menu lateral.
2. Clique em **Edit**, marque **Block all public access** (todas as 4 opções de ACLs e Bucket Policies) e salve.
3. Para aplicar via AWS CLI em uma conta específica:

```bash
aws s3control put-public-access-block \
    --account-id 123456789012 \
    --public-access-block-configuration "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"
```

## 3. Validação e Pontos de Atenção
- Execute `aws s3control get-public-access-block --account-id <ACCOUNT_ID>` e confirme que os 4 parâmetros retornam `true`.
- Ative o **AWS CloudTrail** em todas as regiões para registrar alterações de política e tentativas de login na conta.
