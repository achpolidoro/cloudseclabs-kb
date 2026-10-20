# Preparação de App Registration Somente Leitura para Auditoria (Entra ID e M365)


**Objetivo:** criar no tenant do cliente o acesso somente leitura utilizado pelo coletor de auditoria nas verificações. **Quem executa:** administrador global do cliente, com acompanhamento do analista. **Tempo estimado:** 15 a 20 minutos.

### Sumário

1. Antes de começar
2. Registrar o aplicativo
3. Permissões do Microsoft Graph
4. Exchange Online
5. SharePoint Online
6. Consentimento do administrador
7. Certificado
8. Conferência final
9. O que depende de licença P1/P2
10. Encerramento do acesso

### 1. Registrar o aplicativo no Microsoft Entra ID

1. Acesse **entra.microsoft.com** → **Aplicativos → Registros de aplicativo → Novo registro**.
2. Nome: `M365 Security Assessment Reader`. Tipo de conta: **somente este diretório**. Sem URI de redirecionamento.
3. Anote o **ID do aplicativo (cliente)** e o **ID do diretório (locatário)**.

### 3. Permissões do Microsoft Graph

Em **Permissões de API → Adicionar → Microsoft Graph → Permissões de aplicativo**, adicione:

|Permissão|Para que serve|Exige licença|
|---|---|---|
|Application.Read.All|Aplicações e credenciais|—|
|AuditLog.Read.All|Último acesso, relatório de MFA, entradas e auditoria|Último acesso, MFA e entradas exigem **P1**|
|Device.Read.All|Dispositivos do Entra|—|
|DeviceManagementManagedDevices.Read.All|Conformidade do Intune|Intune|
|Directory.Read.All|Diretório em geral|—|
|Group.Read.All|Grupos e Teams|—|
|IdentityRiskEvent.Read.All|Detecções de risco|**P2**|
|IdentityRiskyUser.Read.All|Usuários em risco|**P2**|
|Organization.Read.All|Organização e licenças|—|
|Policy.Read.All|Acesso Condicional e Security Defaults|—|
|RoleEligibilitySchedule.Read.Directory|Elegibilidade do PIM|**P2**|
|RoleManagement.Read.Directory|Funções administrativas|—|
|SecurityEvents.Read.All|Secure Score|—|
|Sites.Read.All|Endereço do SharePoint|—|
|Team.ReadBasic.All|Equipes do Teams|—|
|User.Read.All|Usuários|—|
|UserAuthenticationMethod.Read.All|Métodos de autenticação|—|

Não é necessário: **Reports.Read.All** (nenhuma verificação usa).

### 4. Exchange Online

1. **Permissões de API → Adicionar → APIs que minha organização usa** → procure **Office 365 Exchange Online** → **Permissões de aplicativo** → `Exchange.ManageAsApp`.
2. **Funções e administradores → Leitor global → Adicionar atribuições** → selecione o aplicativo.

O Exchange exige as duas coisas: a permissão de API **e** uma função (ver README, "Permissão de API × função"). Alternativa sem Leitor global: grupo de funções `View-Only Organization Management` no Exchange (`New-ServicePrincipal` + `Add-RoleGroupMember`).

### 5. SharePoint Online

**Permissões de API → Adicionar → SharePoint → Permissões de aplicativo** → `Sites.FullControl.All`.

**Atenção:** é a API **SharePoint**, no quadro de APIs da Microsoft, e **não** o Microsoft Graph. As duas têm uma permissão com o mesmo nome. No Graph ela não funciona para esta verificação (o SharePoint responde "Unauthorized") e ainda dá escrita em todos os sites. Se ela estiver no Graph, remova.

É a única permissão que a Microsoft aceita, por certificado, para listar a configuração de compartilhamento de todos os sites. A verificação só lê. Função de diretório não tem efeito aqui.

**Alternativa só leitura:** se o cliente não quiser conceder `Sites.FullControl.All`, conceda no **Microsoft Graph → Permissões de aplicativo** só `SharePointTenantSettings.Read.All`. O app lê pelo Graph o compartilhamento externo, o recompartilhamento por convidados e os protocolos legados. Ficam para conferência manual no centro de administração do SharePoint: a lista de compartilhamento por site, o tipo de link padrão, a expiração de link anônimo e o acesso de dispositivos não gerenciados.

### 5.1 Microsoft Teams (verificação complementar "Acesso externo do Teams")

Não precisa de permissão nova: usa `Organization.Read.All` (Graph) e a função **Leitor global**, que já foram configuradas acima. No PC que roda o app, instale o módulo `MicrosoftTeams` (tela **Pré-requisitos**).

Não adicione a API "Skype and Teams Tenant Admin API": segundo a Microsoft, essa permissão causa falha na autenticação por certificado.

### 6. Consentimento do administrador

Em **Permissões de API**, clique em **Conceder consentimento do administrador** e confirme que tudo ficou como **Concedido** (Graph, Exchange e SharePoint).

### 7. Certificado

**Certificados e segredos → Certificados → Carregar certificado** → envie o `.cer` gerado pelo app. Não crie segredo.

### 8. Conferência final

No app: informe o Client ID e clique em **Testar conexão**. O esperado é "Conexão OK" com 17 de 17 permissões. Logo após o consentimento pode levar alguns minutos para as permissões aparecerem.

### 9. O que depende de licença P1/P2

Sem a licença, o app marca o item como **"Não verificado — limitação de licença"** (não é falha) e o relatório registra a limitação.

|Verificação|Exige|
|---|---|
|Último acesso de usuários e convidados (signInActivity)|Entra ID P1 — sem ela o app verifica o resto e marca só o último acesso como indisponível|
|Registro de MFA/SSPR por usuário|Entra ID P1|
|Registros de entrada e autenticação legada|Entra ID P1|
|Usuários e detecções de risco|Entra ID P2|
|Elegibilidade do PIM|Entra ID P2|

### 10. Encerramento do acesso

Ao final do trabalho, o cliente exclui o registro de aplicativo `M365 Security Assessment Reader`. O acesso termina na hora.
