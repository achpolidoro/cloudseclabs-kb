# Publicação segura com Cloudflare Tunnel e DNSSEC

> **Subcategoria:** `infra/cloudflare-dns-tunnels/`

## 1. Expondo serviços sem abrir portas no firewall
Uso do daemon `cloudflared` para conectar serviços locais à borda sem expor o IP público de origem.
