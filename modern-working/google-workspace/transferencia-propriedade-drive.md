# Transferência de Propriedade de Arquivos no Google Drive

> **Caminho:** `modern-working/google-workspace/transferencia-propriedade-drive.md`  
> **Objetivo:** Transferir a propriedade de pastas e arquivos entre usuários do mesmo domínio no Google Workspace preservando a árvore de diretórios.

---

## 1. Pré-requisitos

- Contas de origem e destino ativas dentro do mesmo domínio Google Workspace.
- Permissão para instalação de aplicativos do **Google Workspace Marketplace** (ou transferência via painel Admin).

---

## 2. Passo a passo

### 2.1 Transferência Administrativa via Google Admin (Nativa)

1. Acesse `https://admin.google.com` > **Aplicativos** > **Google Workspace** > **Drive e Documentos**.
2. Clique em **Transferir propriedade**.
3. Informe o e-mail do **Atualmente proprietário** (origem) e do **Novo proprietário** (destino).
4. Clique em **Transferir arquivos**.

### 2.2 Transferência Seletiva de Pastas via Marketplace

Quando for necessário transferir apenas uma pasta específica (e não todo o Drive do usuário):

1. Acesse o aplicativo **Transfer Files Ownership** no Marketplace:
```text
https://workspace.google.com/marketplace/app/transfer_files_ownership/631873519917
```
2. Selecione a pasta de origem no Google Drive, informe o destinatário do mesmo domínio e inicie a transferência recursiva.

---

## 3. Validação e Pontos de Atenção

1. Confirme na conta de destino se os arquivos constam com o novo proprietário na coluna **Proprietário**.
2. A transferência administrativa nativa move todos os arquivos da conta de origem para uma nova pasta identificada com o e-mail antigo na conta de destino.
