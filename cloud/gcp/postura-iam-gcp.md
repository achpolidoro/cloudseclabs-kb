# Postura segura e IAM no GCP

> **Subcategoria:** `cloud/gcp/` • Testado no meu ambiente, assim funcionou.

## 1. Controles essenciais de IAM
- Privilégio mínimo por projeto e pasta.
- Desativação de chaves estáticas de contas de serviço (`iam.disableServiceAccountKeyCreation`).

```bash
gcloud resource-manager org-policies list --project=lab-gcp
```
