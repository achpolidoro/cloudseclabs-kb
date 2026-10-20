# Mapeamento de Superfície Externa com Ferramentas Open Source

> **Caminho:** `cyber/assessment/superficie-open-source.md`  
> **Objetivo:** Identificar subdomínios, portas expostas, certificados TLS e cabeçalhos HTTP de um domínio próprio utilizando ferramentas gratuitas e não intrusivas.

## 1. Pré-requisitos
- Domínio sob sua administração ou com autorização formal de diagnóstico.
- Ambiente Linux/WSL com `subfinder`, `dnsx`, `httpx` (ProjectDiscovery) e `nmap` instalados.

## 2. Passo a passo

### 2.1. Descoberta Passiva de Subdomínios e Resolução DNS
1. Enumere subdomínios através de fontes passivas (Certificate Transparency e bases públicas) sem enviar tráfego direto ao alvo:

```bash
subfinder -d exemplo.com.br -silent | dnsx -silent -a -resp -o subdominios-ativos.txt
```

### 2.2. Inspeção de Serviços HTTP e Certificados TLS
1. Verifique quais subdomínios respondem em HTTP/HTTPS, seus códigos de status, títulos de página e versão de TLS:

```bash
cat subdominios-ativos.txt | cut -d ' ' -f1 | httpx -title -status-code -tech-detect -tls-probe -o relatorio-web.txt
```

2. Verifique portas administrativas expostas indevidamente nos IPs de origem identificados:

```bash
nmap -Pn -sV --top-ports 100 -iL ips-origem.txt -oN scan-portas.txt
```

## 3. Validação e Pontos de Atenção
- Subdomínios órfãos apontando para serviços desativados (registros `CNAME` sem destino ativo) devem ser removidos imediatamente do DNS para evitar *Subdomain Takeover*.
- Painéis administrativos e portas de gerência (`22`, `3389`, `8443`) nunca devem ficar expostos diretamente à internet sem proteção de WAF ou túnel Zero Trust.
