# Diretrizes de Uso Seguro e Sanitização de Prompts em IA Generativa

> **Caminho:** `ia/seguranca-e-automacao/uso-seguro-ia.md`  
> **Objetivo:** Estabelecer práticas de sanitização de dados e configuração de privacidade ao utilizar LLMs em rotinas de engenharia, automação e análise de logs.

## 1. Pré-requisitos
- Acesso às configurações de privacidade da plataforma de IA utilizada.
- Editor de código ou script local para anonimização prévia de artefatos técnicos.

## 2. Passo a passo

### 2.1. Desativação de Treinamento com Dados de Conversa
1. Antes de utilizar qualquer assistente de IA no navegador ou via API, acesse as configurações de **Data Controls / Privacy** da conta.
2. Desative a opção que permite o uso de prompts e arquivos enviados para treinamento de modelos futuros.

### 2.2. Sanitização de Logs, Scripts e Configurações antes do Envio
1. Nunca envie chaves privadas, tokens JWT, strings de conexão, senhas, CPFs, e-mails reais ou nomes de clientes em prompts.
2. Utilize substituição automatizada antes de colar trechos de configuração ou logs para análise:

```powershell
# Exemplo de sanitizacao rapida de log antes de analise externa
(Get-Content .\app.log) `
    -replace '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}', 'usuario@empresa.exemplo' `
    -replace '\b(?:\d{1,3}\.){3}\d{1,3}\b', '192.0.2.10' |
    Set-Content .\app-sanitizado.log
```

## 3. Validação e Pontos de Atenção
- Sempre revise e teste em ambiente isolado qualquer script PowerShell, Bash ou regra de firewall gerada por IA antes de aplicá-la em produção.
- Verifique se pacotes ou bibliotecas sugeridos pelo modelo realmente existem nos repositórios oficiais (prevenção contra *Package Hallucination* / *Slopsquatting*).
