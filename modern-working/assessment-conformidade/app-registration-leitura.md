# Preparação de App Registration Somente Leitura para Auditoria

> **Caminho:** `modern-working/assessment-conformidade/app-registration-leitura.md`  
> **Objetivo:** Configurar um registro de aplicativo (*App Registration*) no Microsoft Entra ID com permissões estritamente de leitura para coleta automatizada de postura de segurança.

---

## 1. Pré-requisitos

- Permissão de **Administrador Global** no Entra ID para conceder o consentimento administrativo (*Admin Consent*).
- Certificado público (`.cer`) gerado na estação do auditor para autenticação *App-Only* sem uso de segredo em texto claro.

---

## 2. Passo a passo

### 2.1 Registrar o Aplicativo no Entra ID

1. Acesse `https://entra.microsoft.com` > **Aplicativos** > **Registros de aplicativo** > **Novo registro**.
2. Defina o nome `M365-Security-Assessment-Reader` e o tipo **Contas somente neste diretório organizacional**.
3. Copie o **ID do aplicativo (Client ID)** e o **ID do diretório (Tenant ID)**.

### 2.2 Adicionar Permissões de API (Microsoft Graph — Application Permissions)

Em **Permissões de API** > **Adicionar uma permissão** > **Microsoft Graph** > **Permissões de aplicativo**, inclua as 17 permissões de leitura abaixo:

| Permissão Graph (Application) | Finalidade na Auditoria | Licença Exigida |
| :--- | :--- | :--- |
| `Organization.Read.All` | Dados do tenant, domínios e licenciamento | — |
| `Directory.Read.All` | Leitura geral de objetos do diretório | — |
| `User.Read.All` | Inventário de usuários e convidados | — |
| `Group.Read.All` | Grupos de segurança e equipes | — |
| `RoleManagement.Read.Directory` | Funções privilegiadas ativas | — |
| `RoleEligibilitySchedule.Read.Directory` | Atribuições elegíveis no PIM | Entra ID P2 |
| `Policy.Read.All` | Acesso Condicional e Security Defaults | — |
| `UserAuthenticationMethod.Read.All` | Métodos de MFA registrados por usuário | Entra ID P1 |
| `AuditLog.Read.All` | Logs de auditoria e último login (`signInActivity`) | Entra ID P1 |
| `IdentityRiskyUser.Read.All` | Usuários sinalizados em risco | Entra ID P2 |
| `IdentityRiskEvent.Read.All` | Detecções de risco de identidade | Entra ID P2 |
| `Application.Read.All` | Enterprise Apps, consentimentos e segredos | — |
| `Device.Read.All` | Dispositivos ingressados/registrados no Entra | — |
| `DeviceManagementManagedDevices.Read.All` | Conformidade de dispositivos no Intune | Intune |
| `SecurityEvents.Read.All` | Pontuação e ações do Microsoft Secure Score | — |
| `Sites.Read.All` | Leitura de configurações do SharePoint | — |
| `Team.ReadBasic.All` | Inventário de equipes do Microsoft Teams | — |

### 2.3 Permissões para Exchange Online e Certificado

1. Em **Permissões de API** > **APIs que minha organização usa**, adicione **Office 365 Exchange Online** > Permissão de aplicativo `Exchange.ManageAsApp`.
2. Atribua a função **Leitor Global** (*Global Reader*) ao aplicativo em **Entra ID** > **Funções e administradores**.
3. Clique em **Conceder consentimento do administrador** e confirme o status verde em todas as permissões.
4. Em **Certificados e segredos** > **Certificados**, faça upload do arquivo `.cer`.

---

## 3. Validação e Pontos de Atenção

1. Gere um par de chaves exclusivo para cada avaliação:
```powershell
$cert = New-SelfSignedCertificate -Subject "CN=M365Assessment-Lab" `
    -CertStoreLocation "Cert:\CurrentUser\My" `
    -KeyExportPolicy Exportable -KeySpec Signature -NotAfter (Get-Date).AddMonths(3)
Export-Certificate -Cert $cert -FilePath ".\M365Assessment-Lab.cer"
```
2. Ao concluir a auditoria, exclua o registro do aplicativo no Entra ID para revogar o acesso imediatamente.
