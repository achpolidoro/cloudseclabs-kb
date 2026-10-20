# Hardening de Servidor OpenSSH e Proteção com Fail2ban no Linux

> **Caminho:** `infra/linux-hardening/ssh-hardening.md`  
> **Objetivo:** Desabilitar autenticação por senha e login direto de `root` no OpenSSH, restringindo o acesso exclusivamente a chaves criptográficas Ed25519 com bloqueio automático de força bruta.

## 1. Pré-requisitos
- Acesso administrativo (`sudo`) ao servidor Linux.
- Chave pública SSH (`~/.ssh/authorized_keys`) já copiada e testada no usuário administrativo.

## 2. Passo a passo

### 2.1. Configuração Segura do Daemon SSH (`sshd_config`)
1. Crie um arquivo de hardening modular em `/etc/ssh/sshd_config.d/99-hardening.conf`:

```text
PermitRootLogin no
PasswordAuthentication no
KbdInteractiveAuthentication no
PubkeyAuthentication yes
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2
X11Forwarding no
```

2. Valide a sintaxe da configuração antes de reiniciar o serviço (mantenha a sessão atual aberta durante o teste):

```bash
sudo sshd -t && sudo systemctl reload ssh
```

### 2.2. Ativação do Fail2ban para a Porta SSH
1. Instale o `fail2ban` e crie `/etc/fail2ban/jail.local`:

```ini
[sshd]
enabled = true
port    = ssh
maxretry = 3
findtime = 10m
bantime  = 24h
```

2. Reinicie o serviço com `sudo systemctl enable --now fail2ban`.

## 3. Validação e Pontos de Atenção
- Abra um **novo terminal** sem fechar a sessão atual e teste o login com sua chave privada (`ssh usuario@ip-servidor`) e confirme que tentativas sem chave retornam `Permission denied (publickey)`.
- Verifique os IPs banidos a qualquer momento com `sudo fail2ban-client status sshd`.
