# Direcionar Usuários do Entra ID para OU Específica no Google

> **Caminho:** `modern-working/identidade-sso/direcionar-ou-google.md`  
> **Objetivo:** Configurar o atributo `OrgUnitPath` no provisionamento automático do Entra ID para criar usuários diretamente em uma Unidade Organizacional (OU) dedicada no Google Workspace.

---

## 1. Pré-requisitos

- Provisionamento automático ativo no aplicativo **Google Cloud / G Suite Connector by Microsoft**.
- Conta **Super Admin** no Google Admin e **Administrador de Aplicativos** no Entra ID.

---

## 2. Passo a passo

### 2.1 Criar a Unidade Organizacional no Google Admin

1. Acesse `https://admin.google.com` > **Diretório** > **Unidades organizacionais**.
2. Crie uma nova OU sob a organização raiz com o nome `Usuarios Entra ID` (evite acentos para prevenir falhas de codificação no mapeamento).

### 2.2 Mapear o Atributo `OrgUnitPath` no Entra ID

1. Acesse `https://entra.microsoft.com` > **Aplicativos empresariais** > **Google Cloud / G Suite Connector by Microsoft**.
2. No menu lateral, clique em **Provisionamento** > **Editar mapeamentos de atributos** > **Provisionar usuários do Microsoft Entra ID**.
3. Clique em **Adicionar novo mapeamento** no final da lista e preencha:

| Campo do Mapeamento | Valor Configurado |
| :--- | :--- |
| **Tipo de mapeamento** | `Constante` |
| **Valor constante** | `/Usuarios Entra ID` |
| **Atributo de destino** | `OrgUnitPath` |
| **Aplicar este mapeamento** | `Somente durante a criação do objeto` |

4. Confirme em **Ok** e clique em **Salvar** no topo.

---

## 3. Validação e Pontos de Atenção

1. Utilize **Provisionar sob demanda** (*Provision on demand*) com um usuário de teste recém-atribuído ao aplicativo.
2. Verifique em **Diretório** > **Usuários** no Google Admin se o usuário foi alocado na OU `/Usuarios Entra ID`.
3. Usuários criados antes desta regra permanecem na OU raiz e devem ser movidos manualmente no Google Admin.
