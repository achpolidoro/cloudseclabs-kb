# Linux e hardening de acesso SSH

> **Subcategoria:** `infra/linux-hardening/` • Ubuntu 24.04 LTS.

```ini
# /etc/ssh/sshd_config.d/99-hardening.conf
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
MaxAuthTries 3
```

```bash
sudo sshd -t && sudo systemctl reload ssh
```
