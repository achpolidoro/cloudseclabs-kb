# Provisionamento Seguro de Instância ARM na Oracle Cloud (OCI)

> **Caminho:** `cloud/oracle-cloud/free-tier-arm.md`  
> **Objetivo:** Criar e blindar uma instância computacional Ampere A1 (ARM) no Always Free Tier da Oracle Cloud com rede isolada e acesso restrito por chave SSH.

## 1. Pré-requisitos
- Conta ativa na Oracle Cloud Infrastructure (OCI).
- Par de chaves SSH Ed25519 gerado localmente (`ssh-keygen -t ed25519`).

## 2. Passo a passo

### 2.1. Configuração de VCN e Security Lists
1. No console OCI, acesse **Networking** > **Virtual Cloud Networks** e inicie o assistente **Start VCN Wizard** (*Create VCN with Internet Connectivity*).
2. Na **Security List** da sub-rede pública, edite a regra de entrada (*Ingress Rule*) da porta `22/TCP` (SSH) para aceitar conexões apenas do seu IP público (`/32`) ou de um túnel privado, removendo `0.0.0.0/0`.

### 2.2. Criação da Instância Ampere A1
1. Acesse **Compute** > **Instances** > **Create instance**.
2. Em **Image and shape**, selecione **Ubuntu Minimal** ou **Oracle Linux** e altere o shape para **Ampere** (`VM.Standard.A1.Flex`, até 4 OCPUs e 24 GB RAM no Free Tier).
3. Em **Add SSH keys**, faça upload da sua chave pública `.pub` e crie a instância.
4. Após conectar via SSH, verifique as regras de firewall locais do sistema operacional (`iptables`/`nftables`):

```bash
sudo iptables -L -n -v
```

## 3. Validação e Pontos de Atenção
- Imagens padrão da OCI utilizam regras `iptables` pré-carregadas além das *Security Lists* da VCN; ao abrir portas na VCN, ajuste também o firewall interno do sistema operacional.
- Ative o **Oracle Cloud Guard** (gratuito) na região principal para monitorar exposições públicas na tenancy.
