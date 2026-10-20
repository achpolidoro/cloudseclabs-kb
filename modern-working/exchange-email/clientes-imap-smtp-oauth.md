# Configuração de Clientes de E-mail com OAuth 2.0 e Autodiscover

> **Caminho:** `modern-working/exchange-email/clientes-imap-smtp-oauth.md`  
> **Objetivo:** Padronizar a configuração de clientes de e-mail com autenticação moderna (OAuth 2.0) e diagnosticar falhas de Autodiscover em redes com DNS interno.

---

## 1. Pré-requisitos

- Autenticação Moderna (*Modern Auth*) habilitada no tenant.
- Registro `CNAME` de `autodiscover` apontando para `autodiscover.outlook.com` (tanto no DNS público quanto no DNS interno, caso haja *Split-Brain DNS*).

---

## 2. Passo a passo

### 2.1 Parâmetros de Servidores e Portas

| Protocolo | Servidor | Porta | Criptografia | Autenticação |
| :--- | :--- | :--- | :--- | :--- |
| **Exchange / MAPI** | Autodiscover automático | `443` | TLS | **OAuth 2.0 (Moderna)** |
| **IMAP** | `outlook.office365.com` | `993` | SSL/TLS | **OAuth 2.0 (Moderna)** |
| **POP3** | `outlook.office365.com` | `995` | SSL/TLS | **OAuth 2.0 (Moderna)** |
| **SMTP** | `smtp.office365.com` | `587` | STARTTLS | **OAuth 2.0 / SMTP AUTH** |

### 2.2 Configuração no Outlook (2016 / 2019 / 365)

1. Abra o Outlook > **Arquivo** > **Adicionar Conta**.
2. Informe o endereço `usuario@seudominio.com.br` e clique em **Conectar**.
3. Conclua a autenticação na janela OAuth (incluindo MFA). Não selecione configuração manual IMAP/POP.

### 2.3 Habilitação de ADAL no Registro (Clientes Legados Outlook 2013)

Caso exista estação legada com Office 2013 que não exiba a tela de login moderno:

```powershell
# Criar chaves DWORD EnableADAL=1 e Version=1 no perfil do usuário
$regPath = "HKCU:\Software\Microsoft\Office\15.0\Common\Identity"
New-Item -Path $regPath -Force | Out-Null
New-ItemProperty -Path $regPath -Name "EnableADAL" -Value 1 -PropertyType DWord -Force
New-ItemProperty -Path $regPath -Name "Version" -Value 1 -PropertyType DWord -Force
```

---

## 3. Validação e Pontos de Atenção

1. Se estações ingressadas no domínio Active Directory falharem ao localizar a caixa, verifique no servidor DNS interno se a zona do domínio possui o registro `CNAME` `autodiscover -> autodiscover.outlook.com`.
2. Clientes de terceiros (ex.: Thunderbird) devem ter o método de autenticação definido como `OAuth2` tanto no servidor de entrada (993) quanto no de saída (587).
