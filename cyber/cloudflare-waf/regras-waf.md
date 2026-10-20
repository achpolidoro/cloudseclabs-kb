# Proteção de borda e WAF na Cloudflare

> **Subcategoria:** `cyber/cloudflare-waf/`

## 1. Regras recomendadas para laboratório
- Bloqueio de países/ASNs fora do escopo de operação em rotas sensíveis.
- Ativação de cabeçalhos de segurança (`CSP`, `X-Content-Type-Options`) e proteção contra *scraping*.
