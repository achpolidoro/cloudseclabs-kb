# Publicação de Serviço Interno sem Abrir Portas com Cloudflare Tunnel

> **Caminho:** `infra/cloudflare-dns-tunnels/cloudflared-tunnel.md`  
> **Objetivo:** Expor uma aplicação web interna com HTTPS e proteção Zero Trust através do daemon `cloudflared`, mantendo todas as portas de entrada (`80/443`) fechadas no firewall de borda.

## 1. Pré-requisitos
- Domínio gerenciado na Cloudflare e acesso ao painel **Cloudflare Zero Trust**.
- Servidor Linux interno com conectividade de saída HTTPS (`443/TCP` e `7844/UDP`) para a rede da Cloudflare.

## 2. Passo a passo

### 2.1. Criação do Túnel no Painel Zero Trust
1. Acesse **Cloudflare Zero Trust** > **Networks** > **Tunnels** > **Create a tunnel**.
2. Selecione **Cloudflared**, defina um nome identificador (ex.: `tunel-lab-interno`) e copie o comando de instalação com o token gerado.

### 2.2. Instalação do Conector e Roteamento de Subdomínio
1. No servidor Linux interno, instale o pacote `cloudflared` e registre o serviço com o token:

```bash
sudo cloudflared service install <TOKEN_DO_TUNEL>
sudo systemctl status cloudflared
```

2. De volta ao painel do túnel, abra a aba **Public Hostname** > **Add a public hostname**:
   - **Subdomain:** `app`
   - **Domain:** `seudominio.com.br`
   - **Service Type:** `HTTP`
   - **URL:** `localhost:8080`

## 3. Validação e Pontos de Atenção
- Proteja o subdomínio recém-publicado criando uma **Access Application** em **Zero Trust** > **Access** > **Applications** para exigir autenticação (OTP por e-mail ou SSO) antes que o tráfego chegue ao servidor interno.
- Confirme no roteador/firewall de borda que nenhum *Port Forwarding* de entrada permanece ativo.
