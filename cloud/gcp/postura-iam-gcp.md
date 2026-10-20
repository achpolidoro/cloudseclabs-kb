# Postura de IAM e Organização Segura no Google Cloud (GCP)

> **Caminho:** `cloud/gcp/postura-iam-gcp.md`  
> **Objetivo:** Aplicar o princípio do menor privilégio no Google Cloud IAM, restringir chaves de Service Accounts e proteger a hierarquia de projetos.

## 1. Pré-requisitos
- Função **Organization Administrator** (`roles/resourcemanager.organizationAdmin`) no Google Cloud.
- `gcloud` CLI instalado e inicializado no terminal.

## 2. Passo a passo

### 2.1. Remoção de Papéis Primitivos e Uso de Grupos
1. No console do GCP, acesse **IAM & Admin** > **IAM** no nível da Organização.
2. Substitua papéis básicos amplos (`Owner`, `Editor`, `Viewer`) atribuídos a contas individuais por papéis predefinidos granulares (ex.: `roles/compute.viewer`, `roles/storage.objectAdmin`) atribuídos a grupos do Google Workspace/Cloud Identity.

### 2.2. Bloqueio de Criação de Chaves de Service Account
1. Acesse **IAM & Admin** > **Organization Policies**.
2. Localize a restrição `iam.disableServiceAccountKeyCreation` (*Disable service account key creation*).
3. Edite a política, selecione **Enforce** e salve para obrigar o uso de **Workload Identity Federation** ou *Service Account Impersonation*:

```bash
gcloud resource-manager org-policies enable-enforce \
    iam.disableServiceAccountKeyCreation \
    --organization=ORGANIZATION_ID
```

## 3. Validação e Pontos de Atenção
- Verifique no **IAM Recommender** permissões não utilizadas nos últimos 90 dias para revogação segura.
- Habilite os *Data Access Audit Logs* para serviços críticos como Cloud KMS, Secret Manager e Cloud Storage.
