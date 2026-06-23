# 🎯 Nmap - Cheatsheet Cyber Sécurité

Guide orienté **reconnaissance réseau** et **énumération** avec Nmap. Focus sur la découverte de services, détection de vulnérabilités et exploitation.

---

## 📖 Qu'est-ce que Nmap ?

**Nmap (Network Mapper)** est l'outil de scanner réseau le plus puissant permettant de :
- Scanner des ports (TCP/UDP)
- Détecter les services et versions
- Identifier les systèmes d'exploitation
- Découvrir des hôtes sur un réseau
- Détecter des vulnérabilités (NSE scripts)
- Cartographier l'infrastructure réseau

**NSE (Nmap Scripting Engine)** : 600+ scripts pour exploitation avancée

---

## 1️⃣ Découverte d'Hôtes

### Ping Scan (Host Discovery)

```bash

#Sans resolution dns
-n

# Ping scan simple
nmap -sn 192.168.1.0/24

# Sans ping (assume tous up)
nmap -Pn 192.168.1.100

# Ping TCP SYN
nmap -PS80,443,22 192.168.1.0/24

# Ping TCP ACK
nmap -PA80,443 192.168.1.0/24

# Ping UDP
nmap -PU53,161 192.168.1.0/24

# Ping ICMP
nmap -PE 192.168.1.0/24         # ICMP Echo
nmap -PP 192.168.1.0/24         # ICMP Timestamp
nmap -PM 192.168.1.0/24         # ICMP Netmask
```

### ARP Scan (réseau local)

```bash
# ARP discovery
nmap -PR 192.168.1.0/24

# Liste des hôtes up
nmap -sn 192.168.1.0/24 | grep "Nmap scan report"
```

---

## 2️⃣ Scan de Ports

### Types de scan TCP

```bash
# SYN Scan (Stealth) - Défaut si root
nmap -sS 192.168.1.100

# Connect Scan (si pas root)
nmap -sT 192.168.1.100

# ACK Scan (firewall detection)
nmap -sA 192.168.1.100

# Window Scan
nmap -sW 192.168.1.100

# NULL Scan
nmap -sN 192.168.1.100

# FIN Scan
nmap -sF 192.168.1.100

# XMAS Scan
nmap -sX 192.168.1.100

# Maimon Scan
nmap -sM 192.168.1.100
```

### Scan UDP

```bash
# UDP Scan (lent)
nmap -sU 192.168.1.100

# Top 20 ports UDP
nmap -sU --top-ports 20 192.168.1.100

# UDP + TCP combiné
nmap -sU -sS 192.168.1.100
```

### Sélection de ports

```bash
# Port spécifique
nmap -p 80 192.168.1.100

# Ports multiples
nmap -p 22,80,443,3389 192.168.1.100

# Range de ports
nmap -p 1-1000 192.168.1.100

# Tous les ports (1-65535)
nmap -p- 192.168.1.100

# Top 100 ports
nmap --top-ports 100 192.168.1.100

# Ports nommés
nmap -p http,https,ssh 192.168.1.100

# Exclure des ports
nmap -p 1-10000 --exclude-ports 80,443 192.168.1.100
```

---

## 3️⃣ Détection de Services et OS

### Version Detection

```bash
# Détection de version
nmap -sV 192.168.1.100

# Intensité de détection (0-9)
nmap -sV --version-intensity 5 192.168.1.100

# Version aggressive (intensité 9)
nmap -sV --version-all 192.168.1.100

# Version light (intensité 2)
nmap -sV --version-light 192.168.1.100
```

### OS Detection

```bash
# Détection OS
nmap -O 192.168.1.100

# OS agressif
nmap -O --osscan-guess 192.168.1.100

# Limite OS detection
nmap -O --max-os-tries 1 192.168.1.100
```

### Scan agressif complet

```bash
# -A = -sV + -O + -sC + traceroute
nmap -A 192.168.1.100

# Avec timing agressif
nmap -A -T4 192.168.1.100
```

---

## 4️⃣ NSE Scripts (Scripting Engine)

### Scripts par défaut

```bash
# Scripts safe par défaut
nmap -sC 192.168.1.100

# Équivalent de
nmap --script=default 192.168.1.100
```

### Scripts par catégorie

```bash
# Vulnérabilités Identification of specific vulnerabilities.
nmap --script vuln 192.168.1.100

# Exploitation This category of scripts tries to exploit known vulnerabilities for the scanned port.
nmap --script exploit 192.168.1.100

# Brute force Executes scripts that try to log in to the respective service by brute-forcing with credentials.
nmap --script brute 192.168.1.100

# Discovery Evaluation of accessible services.
nmap --script discovery 192.168.1.100

# Banner de connexion 
nmap --script banner 192.168.1.100

# Auth 	Determination of authentication credentials.
nmap --script auth 192.168.1.100

# Malware Checks if some malware infects the target system.
nmap --script malware 192.168.1.100

# Intrusive Intrusive scripts that could negatively affect the target system.
nmap --script intrusive 192.168.1.100
```

### Scripts spécifiques

```bash
# Script spécifique
nmap --script http-enum 192.168.1.100

# Multiples scripts
nmap --script http-enum,http-title,http-headers 192.168.1.100

# Wildcard
nmap --script "http-*" 192.168.1.100

# Avec arguments
nmap --script http-brute --script-args userdb=users.txt,passdb=pass.txt 192.168.1.100
```

### Scripts par service

**HTTP/HTTPS**
```bash
nmap -p 80,443 --script http-enum 192.168.1.100
nmap -p 80 --script http-sql-injection 192.168.1.100
nmap -p 80 --script http-shellshock 192.168.1.100
nmap -p 80 --script http-vuln-* 192.168.1.100
nmap -p 443 --script ssl-* 192.168.1.100
```

**SMB**
```bash
nmap -p 445 --script smb-enum-shares 192.168.1.100
nmap -p 445 --script smb-enum-users 192.168.1.100
nmap -p 445 --script smb-os-discovery 192.168.1.100
nmap -p 445 --script smb-vuln-* 192.168.1.100
```

**SSH**
```bash
nmap -p 22 --script ssh-auth-methods 192.168.1.100
nmap -p 22 --script ssh-brute 192.168.1.100
nmap -p 22 --script ssh-hostkey 192.168.1.100
```

**FTP**
```bash
nmap -p 21 --script ftp-anon 192.168.1.100
nmap -p 21 --script ftp-brute 192.168.1.100
nmap -p 21 --script ftp-bounce 192.168.1.100
```

**SQL**
```bash
nmap -p 3306 --script mysql-enum 192.168.1.100
nmap -p 3306 --script mysql-brute 192.168.1.100
nmap -p 1433 --script ms-sql-info 192.168.1.100
nmap -p 5432 --script pgsql-brute 192.168.1.100
```

**RDP**
```bash
nmap -p 3389 --script rdp-enum-encryption 192.168.1.100
nmap -p 3389 --script rdp-vuln-ms12-020 192.168.1.100
```

**DNS**
```bash
nmap -p 53 --script dns-zone-transfer -script-args dns-zone-transfer.domain=target.com 192.168.1.100
nmap -p 53 --script dns-brute --script-args dns-brute.domain=target.com 192.168.1.100
```

---

## 5️⃣ Timing et Performance

### Templates de timing

```bash
# T0 - Paranoïaque (IDS evasion)
# T1 - Sneaky (IDS evasion)
# T2 - Polite (moins de bande passante)
# T3 - Normal (défaut)
# T4 - Aggressive (réseau rapide)
# T5 - Insane (très rapide, perte de paquets)
nmap -T5 192.168.1.100
```

### Contrôle fin du timing

```bash
# Host timeout
nmap --host-timeout 10m 192.168.1.100

# Min/Max rate
nmap --min-rate 100 192.168.1.100
nmap --max-rate 1000 192.168.1.100

# Parallélisme
nmap --min-parallelism 100 192.168.1.100
nmap --max-parallelism 256 192.168.1.100

# Retries
nmap --max-retries 3 192.168.1.100

# Scan delay
nmap --scan-delay 1s 192.168.1.100
nmap --max-scan-delay 10s 192.168.1.100
```

---

## 6️⃣ Évasion de Firewall/IDS

### Fragmentation

```bash
# Fragmentation simple
nmap -f 192.168.1.100

# MTU custom
nmap --mtu 16 192.168.1.100
```

### Decoy (leurres)

```bash
# Decoy addresses
nmap -D RND:10 192.168.1.100

# Decoys spécifiques
nmap -D 192.168.1.5,192.168.1.6,ME,192.168.1.8 192.168.1.100
```

### Spoofing

```bash
# Spoof source IP
nmap -S 192.168.1.50 192.168.1.100

# Spoof MAC
nmap --spoof-mac 0 192.168.1.100
nmap --spoof-mac Apple 192.168.1.100
nmap --spoof-mac 00:11:22:33:44:55 192.168.1.100
```

### Options avancées

```bash
# Data length aléatoire
nmap --data-length 25 192.168.1.100

# TTL custom
nmap --ttl 64 192.168.1.100

# Randomize hosts
nmap --randomize-hosts 192.168.1.0/24

# Bad checksum
nmap --badsum 192.168.1.100

--stats-every=5s	Shows the progress of the scan every 5 seconds.
```

## 7️⃣ Output et Reporting

### Formats de sortie

```bash
# Normal output
nmap -oN scan.txt 192.168.1.100

# XML output
nmap -oX scan.xml 192.168.1.100

# Greppable output
nmap -oG scan.gnmap 192.168.1.100

# Script kiddie output
nmap -oS scan.txt 192.168.1.100

# All formats
nmap -oA scan 192.168.1.100
```

### Verbosité

```bash
# Verbose
nmap -v 192.168.1.100

# Very verbose
nmap -vv 192.168.1.100

# Debugging
nmap -d 192.168.1.100
nmap -dd 192.168.1.100  # Plus de debug

# Packet trace
nmap --packet-trace 192.168.1.100

# Raisons des états
nmap --reason 192.168.1.100

# Open ports seulement
nmap --open 192.168.1.100
```

---

## 8️⃣ Scans Réseau Complets

### Scan rapide

```bash
# Top 100 ports, version, rapide
nmap -sS -sV -T4 --top-ports 100 192.168.1.0/24
```

### Scan complet d'un hôte

```bash
# All ports, version, OS, scripts, aggressive
nmap -sS -sV -sC -O -p- -T4 -A 192.168.1.100 -oA full-scan
```

### Scan de réseau

```bash
# Discovery + version sur réseau
nmap -sS -sV -T4 -F 192.168.1.0/24 -oG network-scan.gnmap
```

### Scan vulnérabilités

```bash
# Vuln detection complète
nmap -sS -sV --script vuln -T4 192.168.1.100 -oA vuln-scan
```

---

## 9️⃣ Scripts NSE Essentiels

### Énumération Web

```bash
# HTTP enumeration
nmap -p 80,443 --script http-enum,http-title,http-headers,http-methods 192.168.1.100

# Directory brute force
nmap -p 80 --script http-brute-dirs --script-args http-brute-dirs.path=/admin 192.168.1.100

# SQL injection detection
nmap -p 80 --script http-sql-injection 192.168.1.100

# XSS detection
nmap -p 80 --script http-csrf,http-stored-xss,http-dombased-xss 192.168.1.100
```

### Brute Force

```bash
# SSH brute force
nmap -p 22 --script ssh-brute --script-args userdb=users.txt,passdb=pass.txt 192.168.1.100

# FTP brute force
nmap -p 21 --script ftp-brute --script-args userdb=users.txt,passdb=pass.txt 192.168.1.100

# SMB brute force
nmap -p 445 --script smb-brute --script-args userdb=users.txt,passdb=pass.txt 192.168.1.100

# MySQL brute force
nmap -p 3306 --script mysql-brute --script-args userdb=users.txt,passdb=pass.txt 192.168.1.100
```

### Vulnérabilités connues

```bash
# EternalBlue (MS17-010)
nmap -p 445 --script smb-vuln-ms17-010 192.168.1.100

# Shellshock
nmap -p 80 --script http-shellshock --script-args uri=/cgi-bin/test.sh 192.168.1.100

# Heartbleed
nmap -p 443 --script ssl-heartbleed 192.168.1.100

# BlueKeep (RDP)
nmap -p 3389 --script rdp-vuln-ms12-020 192.168.1.100
```

---

## 🔟 Exploitation Post-Scan

### Extraction de données

```bash
# Extraire les ports ouverts
grep "open" scan.gnmap | cut -d' ' -f2 | sort -u

# Extraire IPs avec port 80 ouvert
grep "80/open" scan.gnmap | cut -d' ' -f2

# Compter les hôtes up
grep "Status: Up" scan.gnmap | wc -l

# Services par port
grep "open" scan.gnmap | cut -d' ' -f3 | sort | uniq -c | sort -nr
```

### Conversion XML

```bash
# XML vers HTML
xsltproc scan.xml -o scan.html

# Avec Nmap stylesheet
xsltproc /usr/share/nmap/nmap.xsl scan.xml -o scan.html
```

---

## 1️⃣1️⃣ Scans par Scénario

### Pentest externe

```bash
# 1. Discovery
nmap -sn target-ranges.txt -oG hosts-up.txt

# 2. Port scan rapide
nmap -iL hosts-up.txt -sS -T4 --top-ports 1000 -oA quick-scan

# 3. Full scan sur hôtes intéressants
nmap -sS -sV -sC -O -p- -T4 interesting-hosts.txt -oA full-scan

# 4. Vuln scan
nmap -sS -sV --script vuln interesting-hosts.txt -oA vuln-scan
```

### Pentest interne

```bash
# 1. ARP discovery
nmap -sn -PR 192.168.1.0/24 -oG internal-hosts.txt

# 2. Full TCP scan
nmap -sS -sV -sC -p- -T4 192.168.1.0/24 -oA internal-full

# 3. UDP top ports
nmap -sU --top-ports 100 192.168.1.0/24 -oA internal-udp

# 4. SMB/AD enumeration
nmap -p 445 --script smb-enum-* 192.168.1.0/24
```

### Web Application

```bash
# Scan web complet
nmap -p 80,443,8080,8443 -sV -sC --script "http-*" target.com -oA web-scan

# Avec vulns
nmap -p 80,443 --script http-enum,http-vuln-* target.com
```

### Database Enumeration

```bash
# MySQL
nmap -p 3306 -sV --script mysql-* 192.168.1.100

# MSSQL
nmap -p 1433 -sV --script ms-sql-* 192.168.1.100

# PostgreSQL
nmap -p 5432 -sV --script pgsql-* 192.168.1.100

# MongoDB
nmap -p 27017 -sV --script mongodb-* 192.168.1.100
```

---


## 💡 Tips Pro

1. **Toujours commencer par un ping scan** (-sn) pour découverte
2. **-sS + -sV + -T4** pour scan rapide et efficace
3. **--top-ports 1000** pour coverage rapide
4. **-p-** pour scan complet (mais long)
5. **NSE scripts** pour exploitation avancée
6. **-oA** pour tous les formats de sortie
7. **--reason** pour comprendre les états des ports
8. **Combiner -sU avec -sS** pour TCP + UDP
9. **--script vuln** pour détection automatique de vulns
10. **Timing T4** pour la plupart des scans (sauf stealth)

---

**🎯 Nmap est l'outil de reconnaissance réseau le plus puissant. Maîtrisez-le pour cartographier n'importe quelle infrastructure !**
