# Configuração de Clientes de E-mail (Exchange, IMAP, POP e SMTP com OAuth)

## 1. Parâmetros de Conexão e Criptografia

| Protocolo | Servidor | Porta | Criptografia | Autenticação |
| :--- | :--- | :--- | :--- | :--- |
| **IMAP** | `outlook.office365.com` | `993` | SSL/TLS | **OAuth 2.0 (Moderna)** |
| **POP3** | `outlook.office365.com` | `995` | SSL/TLS | **OAuth 2.0 (Moderna)** |
| **SMTP** | `smtp.office365.com` | `587` | STARTTLS | **OAuth 2.0 / SMTP AUTH** |

> **Importante:** A autenticação básica (*Basic Auth*) para IMAP e POP está desativada por padrão. Clientes de e-mail devem suportar **OAuth 2.0** ou autodiscover moderno.

---

## 2. Configuração por Versão do Cliente

### Outlook 2016 / 2019 / M365
1. Abra o Outlook → **Arquivo** → **Adicionar Conta**.
2. Informe o endereço completo (`usuario@seudominio.com.br`) e clique em **Conectar**.
3. Conclua o login na janela de autenticação moderna (incluindo MFA).
4. Não selecione IMAP/POP manualmente — permita que o **Autodiscover** configure via protocolo nativo.

### Ajuste de Registro em Clientes Legados (Outlook 2013)
> Versão fora de suporte oficial; requer habilitação manual do ADAL no Registro do Windows:

1. Feche o Outlook e abra o `regedit`.
2. Acesse `HKEY_CURRENT_USER\Software\Microsoft\Office\15.0\Common\Identity`.
3. Crie dois valores `DWORD (32 bits)`:
   - `EnableADAL` = `1`
   - `Version` = `1`
4. Reinicie a estação e recrie o perfil pelo Painel de Controle (**Mail**).

---

## 3. Verificação de DNS Interno (Split-Brain DNS)

Em redes corporativas com Active Directory local usando o mesmo domínio de e-mail (`seudominio.com.br`), a falha no Autodiscover geralmente ocorre por ausência do registro `CNAME` na zona DNS interna.

- **Solução no DNS interno:** Criar o registro `CNAME` apontando `autodiscover` para `autodiscover.outlook.com`.

## 4. Diagnóstico de TLS e Conectividade SMTP
- **MxToolbox SMTP Diagnostics:** `https://mxtoolbox.com/diagnostic.aspx` (validação de banner, TLS e listas de bloqueio).
- **CheckTLS:** `https://www.checktls.com/TestReceiver` (inspeção de certificado e cifra TLS).
