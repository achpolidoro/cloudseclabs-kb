# Uso de aliases com “+” no Microsoft 365


No Microsoft 365 (Exchange Online) existe suporte para **plus addressing**—também chamado de subadresseamento ou endereços com “+”. Isso significa que, sim, é possível usar aliases do tipo `usuario+1@meudominio.com`.

## O que é o plus addressing

- É um método de criar variações de um mesmo endereço de email sem precisar cadastrar manualmente cada alias.
- Tudo que entrar em `usuario+qualquercoisa@meudominio.com` será entregue na caixa de entrada de `usuario@meudominio.com`.

## Como ativar no Exchange Online

1. Conecte-se ao Exchange Online via PowerShell.
2. Execute o cmdlet para habilitar globalmente:
    
    ```powershell
    Set-OrganizationConfig -AllowPlusAddressInRecipients $true
    
    ```
    
3. Pronto. Usuários podem receber e-mails enviados para `usuario+balanco@meudominio.com`, por exemplo.

## Principais observações

- A configuração é feita por inquilino (tenant), não por usuário.
- Plus addressing não cria entradas no Global Address List (GAL). Todos chegam à conta principal.
- Assim como outros aliases, não há custo extra nem consumo de licenças.
- Compatibilidade de envio: ao responder, dependendo do cliente, o remetente poderá escolher usar o alias com “+” (no Outlook na Web) ou apenas o SMTP padrão.

---

Mais sobre recursos de gerenciamento de email no Microsoft 365:

- Enviar e-mails de diferentes aliases: Outlook na Web pode exibir um menu “De” com todos os proxies habilitados.
- Gerenciar até 400 aliases tradicionais por usuário (sem “+”) via **Centro de Administração** ou PowerShell.
- Dicas avançadas para regras de transporte e roteamento em https://learn.microsoft.com/microsoft-365/
