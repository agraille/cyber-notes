# Hardening Linux

## 1. Le concept

### Où ça se situe dans NIST

Comme pour Windows, le durcissement Linux relève de la fonction **Protect (PR)** du NIST CSF — catégories **PR.AC (Access Control)** pour la partie authentification, et **PR.PT (Protective Technology)** pour le confinement des applications et la traçabilité. Le principe directeur reste le même : réduire ce qu'un attaquant peut atteindre, et tracer ce qu'il fait s'il y parvient malgré tout.

### C'est quoi concrètement

Le durcissement Linux repose sur trois piliers complémentaires :
- Sécuriser l'accès distant (SSH), point d'entrée le plus exposé sur un serveur Linux
- Confiner les processus (AppArmor ou SELinux) pour limiter les dégâts en cas de compromission d'une application
- Tracer l'activité système (auditd) pour permettre la détection et l'investigation

## 2. Pourquoi ça marche (le mécanisme)

**SSH par mot de passe** est vulnérable au bruteforce : un attaquant peut essayer des combinaisons indéfiniment tant qu'il n'est pas bloqué. L'authentification par clé élimine ce risque puisqu'il n'y a plus de secret devinable par force brute.

**Le confinement applicatif (AppArmor/SELinux)** répond à une réalité simple : même une application bien maintenue peut contenir une faille inconnue. Sans confinement, un processus compromis (par exemple un serveur web exploité) hérite de tous les droits de l'utilisateur qui l'exécute, et peut lire/écrire n'importe où sur le système. Un profil de confinement définit explicitement ce qu'un processus a le droit de faire (quels fichiers lire, quels réseaux joindre), donc même exploité, il reste enfermé dans ce périmètre — c'est le principe du **moindre privilège appliqué au niveau processus**, pas seulement au niveau utilisateur.

**auditd** ne prévient rien en lui-même, mais transforme des actions invisibles (une modification de `/etc/passwd`, un usage de `sudo`) en événements journalisés exploitables pour la détection — sans cette instrumentation, ces actions ne laissent aucune trace.

## 3. Mise en œuvre — le chemin concret

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
