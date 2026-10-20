# Hardening e Proteção de Dispositivos Móveis (iOS e Android)

> **Caminho:** `mobile/protecao-dispositivos/hardening-mobile.md`  
> **Objetivo:** Configurar camadas nativas de proteção contra roubo de aparelho, sequestro de sessão, SIM Swap e redes Wi-Fi hostis em smartphones iOS e Android.

## 1. Pré-requisitos
- Dispositivo iOS (17.3 ou superior) ou Android (14 ou superior) atualizado.
- Conta principal protegida com autenticação multifator (MFA).

## 2. Passo a passo

### 2.1. Proteção de Dispositivo Roubado e Bloqueio de Tela
1. **No iOS:** Acesse **Ajustes** > **Face ID e Código** e ative **Proteção de Dispositivo Roubado** (*Stolen Device Protection*) em modo **Sempre**, exigindo biometria com atraso de segurança para alterações de conta Apple ou chaves salvas. Desative também o acesso à **Central de Controle** e **Acessórios USB** na tela bloqueada.
2. **No Android:** Acesse **Configurações** > **Segurança e privacidade** > **Proteção contra roubo** e ative **Bloqueio por detecção de roubo** e **Bloqueio de dispositivo offline**.

### 2.2. PIN de Chip (SIM/eSIM) e DNS Criptografado
1. Ative o **PIN do SIM** (alterando o padrão da operadora) ou migre para **eSIM**, impedindo que o chip físico seja movido para outro aparelho para interceptar códigos SMS.
2. Configure DNS sobre HTTPS/TLS nativo no sistema (ex.: `one.one.one.one` ou `dns.quad9.net`) em **DNS Privado** (Android) ou perfil de configuração DNS (iOS).

## 3. Validação e Pontos de Atenção
- Utilize aplicativos autenticadores com bloqueio biométrico próprio e evite depender de SMS como segundo fator em contas de e-mail ou bancos.
- Revise periodicamente os dispositivos conectados à conta Google/Apple e revogue sessões antigas.
