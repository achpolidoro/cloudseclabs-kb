# Administração Básica do Microsoft 365 (Exchange Online e Entra Connect)

Este guia reúne os procedimentos básicos para consulta de usuários, gerenciamento de licenças do **Exchange Online Plan 1** e boas práticas de operação com o **Active Directory (AD) local** sincronizado.


---

## 1. Acesso ao Painel Administrativo

* **Endereço de acesso:** Acesse o portal pelo link `admin.cloud.microsoft`.
* **Conta de acesso:** Utilize sempre a conta exclusiva de **Administrador Global** para gerenciar o ambiente, e não a conta de e-mail de uso diário.
* **Boas práticas de segurança:** Mantenha a conta administrativa separada do uso rotineiro, sem licença de e-mail atribuída, armazenando a credencial em um cofre de senhas e utilizando autenticação multifator (Authenticator).


---

## 2. Entendendo a Sincronização (AD Local vs. Nuvem)

* **Quem manda nos usuários:** Para padronizar e centralizar a gestão, o ambiente está sincronizado com o seu Active Directory (AD) local.
* **Regra de ouro:** Todo usuário sincronizado deve ser criado, editado ou excluído **diretamente no AD local**, pois o painel online não permitirá apagar ou manter alterações que divirjam do servidor local.
* **Tempo de sincronização:** O sincronismo de usuários e hashes de senha ocorre automaticamente em intervalos de aproximadamente **30 minutos**.
* **Monitoramento de erros:** Na tela inicial (**Home**) do painel `admin.cloud.microsoft`, você pode conferir o horário da última sincronização. Caso haja algum problema (como dois usuários criados no AD com o mesmo endereço/alias de e-mail), o status ficará **vermelho** indicando o erro de sincronização.
* **Como saber se um usuário veio do AD Local:** No painel do **Microsoft Entra ID**, verifique a coluna `onPremiseSyncEnabled`. Todos os usuários marcados como **Yes** existem obrigatoriamente no seu AD local. Ao clicar no usuário e acessar **Propriedades**, o painel exibe a Unidade Organizacional (OU) exata onde ele está localizado e o horário da última sincronização.


---

## 3. Como Consultar Usuários, Licenças e Armazenamento

### 3.1. Verificar o total de licenças contratadas e disponíveis


1. No menu lateral esquerdo do `admin.cloud.microsoft`, clique em **Cobrança (Billing)** > **Licenças (Licenses)**.
2. Clique sobre o produto **Exchange Online (Plan 1)**.
3. O painel mostrará o total de licenças contratadas, quantas estão atribuídas no momento e quantas estão disponíveis para uso (ex.: 55 contratadas).

### 3.2. Listar apenas os usuários que consomem licença


1. No menu lateral esquerdo, acesse **Usuários (Users)**.
2. Utilize a opção de **Filtro** na lista para exibir apenas os usuários **Licenciados**.
3. Caso precise analisar a relação fora do painel, utilize a opção de **Exportar** a lista.

### 3.3. Verificar o uso de armazenamento (Cota de 50 GB)


1. Na lista de usuários do painel, clique sobre o usuário desejado.
2. Acesse a guia **E-mail (Mail)**.
3. O painel exibirá quantos GB estão sendo utilizados do limite de **50 GB** da caixa de correio (havendo necessidade futura por excesso de uso, é possível avaliar a ativação de mais 50 GB de arquivo morto).


---

## 4. Como Criar uma Nova Conta de E-mail e Atribuir Licença

*(Procedimento aplicável tanto para novos colaboradores quanto para caixas de setor onde vários usuários precisam responder utilizando o endereço institucional do departamento, já que listas de distribuição enviam a resposta com o e-mail individual do remetente)*.


1. **Criar no AD Local:** Acesse o seu Active Directory local, crie o usuário dentro de uma Unidade Organizacional (OU) que já esteja sincronizada e preencha o campo de **E-mail**.
2. **Aguardar a Sincronização:** Aguarde cerca de **30 minutos** para que a conta apareça na listagem de usuários do `admin.cloud.microsoft`.
3. **Atribuir a Licença:**
   * No painel `admin.cloud.microsoft`, clique sobre o novo usuário na lista.
   * Acesse a guia **Licenças e aplicativos (Licenses)**.
   * Marque a caixa da licença **Exchange Online (Plan 1)** e clique em **Salvar**.
4. **Aguardar a Criação da Caixa (Provisionamento):**
   * Após salvar a licença, a Microsoft leva cerca de **15 a 20 minutos** para criar a caixa de correio.
   * **Dica de verificação:** Clique no usuário e vá até a guia **E-mail**. Enquanto a caixa não estiver pronta, aparecerá a mensagem informando que *"a criação da caixa de correio está em progresso"*. Assim que a mensagem sumir e exibir os dados da caixa, o acesso estará liberado.
5. **Conferir no Sophos:** Somente após o usuário sincronizar e a caixa estar criada no Microsoft 365, acesse o painel do **Sophos** para validar a vinculação da conta e as regras de recebimento de e-mails.


---

## 5. Como Remover Licenças e Excluir Contas com Segurança

### 5.1. Passo a passo para remover uma licença


1. Antes de remover a licença de contas antigas ou de teste, abra a conta ou verifique a guia **E-mail** para confirmar se não há uso ativo ou envio de alertas/notificações de sistemas (como ERP).
2. Clique sobre o usuário no painel `admin.cloud.microsoft` e acesse a guia **Licenças e aplicativos**.
3. **Desmarque** a opção **Exchange Online (Plan 1)** e clique em **Salvar**.
4. A licença será liberada imediatamente e já aparecerá como disponível em **Cobrança > Licenças**.

### 5.2. Cuidados importantes após remover a licença

* **Remoção em duas etapas (Recomendado):** Quando não tiver certeza se uma conta é utilizada por algum processo automático, primeiro **apenas remova a licença** e aguarde alguns dias. Se nenhum serviço apresentar falha, prossiga com a exclusão da conta.
* **Prazo de retenção da caixa (30 dias):** Quando você remove a licença de um usuário, a caixa de correio permanece retida na Microsoft por **30 dias** e, após esse prazo, é apagada definitivamente.
* **Como manter os e-mails sem gastar licença (Caixa Compartilhada):** Se precisar liberar a licença de um usuário desligado ou antigo sem perder os e-mails dele após 30 dias, converta a caixa em **Caixa Compartilhada** (que não exige licença para até 50 GB) e delegue um usuário licenciado ativo como responsável (com permissão de ler/enviar em nome da caixa) para garantir que ela seja mantida no ambiente.

### 5.3. Exclusão e Restauração de Usuários

* **Como excluir:** Para usuários vindos do AD, faça a exclusão diretamente no **Active Directory local**; no próximo ciclo de sincronização, ele será removido da lista ativa da nuvem. Contas criadas exclusivamente na nuvem (sem vínculo com o AD) podem ser excluídas direto pelo painel online.
* **Como restaurar em caso de exclusão por engano:**
  * Certifique-se de que a **Lixeira do Active Directory** esteja habilitada no seu servidor local para permitir recuperar objetos apagados acidentalmente.
  * No painel online (em **Usuários > Usuários deletados / Deleted users**), a conta excluída fica retida por **30 dias**.
  * **Atenção:** Se o usuário era sincronizado com o AD local, a restauração **precisa ser feita no AD local**. Caso você clique em restaurar apenas no painel online, na próxima sincronização (em 30 minutos) o AD local mandará apagar o usuário novamente.