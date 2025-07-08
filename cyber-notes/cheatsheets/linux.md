# 🐧 Linux Cheatsheet – Commandes essentielles

## 📂 Navigation système

pwd  
> Affiche le chemin absolu du répertoire courant.  

ls -la  
> Liste tous les fichiers (même cachés) avec détails.  

cd /chemin/vers/dossier  
> Change de répertoire.

mkdir nouveau_dossier  
> Crée un nouveau dossier.  

rm fichier  
> Supprime un fichier.  

rm -r dossier  
> Supprime un dossier et son contenu.  

cp fichier1 fichier2  
> Copie un fichier.  

mv source destination  
> Déplace ou renomme un fichier/dossier.

---

## 🔍 Recherche de fichiers

find / -name "fichier"  
> Recherche un fichier à partir de la racine.  

locate fichier  
> Recherche rapide dans la base de données pré-indexée.  

updatedb  
> Met à jour la base de `locate`.

grep "texte" fichier  
> Recherche une chaîne de texte dans un fichier.  

grep -r "texte" dossier  
> Recherche récursive dans les fichiers du dossier.

---

## ⚙️ Gestion des processus

ps aux  
> Affiche tous les processus en cours.

htop  
> Version améliorée de `top` (si installé).  

kill PID  
> Termine un processus par son ID.  

kill -9 PID  
> Forçage si `kill` simple échoue.  

pkill nom  
> Termine tous les processus avec le nom donné.

---

## 🔐 Permissions et utilisateurs

chmod +x script.sh  
> Rend un fichier exécutable.  

chmod 755 fichier  
> Définit les permissions en numérique.  

chown user:group fichier  
> Change le propriétaire du fichier.  

whoami  
> Affiche l’utilisateur courant.  

id  
> Affiche UID, GID et groupes.

---

## 📦 Gestion des paquets

### Debian / Ubuntu (apt)

apt update  
> Met à jour la liste des paquets.  

apt upgrade  
> Met à jour les paquets installés.  

apt install nom  
> Installe un paquet.  

apt remove nom  
> Supprime un paquet.

---

## 🧠 Informations système

uname -a  
> Infos système complètes.  

df -h  
> Affiche l’espace disque utilisé/disponible.  

du -sh *  
> Taille de chaque fichier/dossier dans le répertoire courant.  

free -h  
> Mémoire RAM utilisée/disponible.  

uptime  
> Temps depuis le dernier redémarrage.  

who
> Liste les utilisateurs connectés.  

w  
> Activité des utilisateurs.

---
---
