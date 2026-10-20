# Validação prática de SPF, DKIM e DMARC

> **Subcategoria:** `cyber/email-dmarc/` • Testado no meu ambiente, assim funcionou.

## 1. Consultando registros via terminal
```bash
dig +short TXT cloudseclabs.com.br
dig +short TXT _dmarc.cloudseclabs.com.br
```

## 2. Tabela de referência
| Protocolo | Host | Valor | Finalidade |
| :--- | :--- | :--- | :--- |
| **SPF** | `@` | `v=spf1 include:_spf.google.com -all` | Servidores autorizados |
| **DMARC** | `_dmarc` | `v=DMARC1; p=reject; adkim=s; aspf=s` | Política de rejeição |
