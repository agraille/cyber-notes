# Hardening Linux

## C'est quoi

Le durcissement Linux repose sur trois piliers :

1. **Sécuriser SSH** (point d'entrée le plus exposé) — clés plutôt que mots de passe, limiter les tentatives
2. **Confiner les processus** (AppArmor/SELinux) — limiter ce qu'un programme compromis peut faire
3. **Tracer l'activité** (auditd) — transforme actions invisibles en événements détectables

**Enjeux** :
- **SSH par mot de passe** = bruteforce possible, clés = sûr
- **Pas de confinement** = une app web exploitée = accès root possible
- **Pas de traçabilité** = compromission invisible

---

## Mise en œuvre — le chemin concret

### Durcir l'accès SSH

Configuration recommandée dans `/etc/ssh/sshd_config` :
```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
MaxAuthTries 3
AllowUsers deploy admin
ClientAliveInterval 300
ClientAliveCountMax 2
X11Forwarding no
```

Restreindre les algorithmes de chiffrement faibles :
```
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com
KexAlgorithms curve25519-sha256@libssh.org
MACs hmac-sha2-512-etm@openssh.com
```

```bash
sshd -t              # test de syntaxe avant reload
systemctl reload sshd
```

Fail2ban en couche supplémentaire contre le bruteforce (utile même avec l'authentification par clé désactivée pour les mots de passe, en cas de mauvaise config sur un autre service) :
```bash
apt install fail2ban
# /etc/fail2ban/jail.local
[sshd]
enabled = true
maxretry = 4
bantime = 3600
```

### Confiner avec AppArmor (Debian/Ubuntu)

```bash
aa-status                                  # lister les profils actifs et leur mode
aa-enforce /etc/apparmor.d/usr.sbin.nginx  # passer un profil en mode bloquant (enforce)
aa-genprof /usr/bin/monapp                 # générer un profil pour une application non couverte
```

### Confiner avec SELinux (RHEL/CentOS)

```bash
getenforce
sestatus
```

Rendre le mode enforcing persistant dans `/etc/selinux/config` :
```
SELINUX=enforcing
```

Analyser les refus SELinux pour ajuster une policy sans désactiver la protection (plutôt que de céder à la tentation de tout désactiver quand ça bloque une application) :
```bash
ausearch -m avc -ts recent
sealert -a /var/log/audit/audit.log
```

### Tracer les actions sensibles avec auditd

```bash
apt install auditd

# Tracer toute modification de /etc/passwd et /etc/shadow
auditctl -w /etc/passwd -p wa -k identity_changes
auditctl -w /etc/shadow -p wa -k identity_changes

# Tracer l'usage de sudo
auditctl -w /etc/sudoers -p wa -k sudo_changes
```

```bash
ausearch -k identity_changes
```

## Documentation officielle

- NIST Cybersecurity Framework : https://www.nist.gov/cyberframework
- OpenSSH hardening : https://www.ssh.com/academy/ssh/sshd_config
- AppArmor : https://gitlab.com/apparmor/apparmor/-/wikis/Documentation
- SELinux : https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/using_selinux/
- CIS Benchmarks Linux : https://www.cisecurity.org/cis-benchmarks
