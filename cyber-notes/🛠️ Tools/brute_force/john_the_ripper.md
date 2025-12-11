# 🔐 John The Ripper - Cheatsheet CyberSécurité

John the Ripper (JtR) est un outil de **crackage de mots de passe**, compatible avec des fichiers `/etc/shadow`, des dumps de hash (NTLM, MD5, bcrypt...), et bien plus encore.

---

## 🧰 1. Détection automatique

john hash.txt  
> Crackage automatique avec format détecté

john --format=raw-md5 hash.txt  
> Crackage en précisant le format

---

## 🛠 2. Préparer les hashs

### 🔎 Identifier le type de hash

john --list=formats  
> Affiche tous les formats supportés

$1$ = MD5
$2y$ = bcrypt
$6$ = SHA512


### 📎 Convertir vers un format supporté

- Utiliser `unshadow` pour combiner `/etc/passwd` + `/etc/shadow` :
  
  unshadow passwd.txt shadow.txt > hash.txt

- Pour Windows : extraire les hashs NTLM avec `pwdump` ou `impacket-secretsdump`.

---

## 📂 3. Wordlists

john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt  
> Crackage par dictionnaire

john --incremental hash.txt  
> Bruteforce total (long mais efficace)

---

## 📊 4. Voir la progression

john --status  
> Affiche l’état actuel du cracking

john --show hash.txt  
> Affiche les mots de passe trouvés

---

## 🔁 5. Reprendre une session

john --session=mySession --wordlist=rockyou.txt hash.txt  
> Lance une session nommée

john --restore=mySession  
> Reprend une session arrêtée

---

## 🧪 6. Exemples de format

### 🔐 /etc/shadow (Linux)

root:$6$123456$EXAMPLEHASH:...
> SHA-512 Linux → `--format=sha512crypt`

### 🪟 Hash NTLM Windows

Administrator:500:aad3b435b51404eeaad3b435b51404ee:HASH:...
> NTLM → `--format=nt`

### 🔒 Hash MD5 brut

HASH  
> MD5 simple → `--format=raw-md5`

---

## 💣 7. Générer des hashs (pour test)

echo -n "password" | openssl passwd -1 -stdin  
> Hash MD5 crypt

mkpasswd --method=sha-512  
> Hash SHA512 Linux (via `whois`)

---

## 🛡 8. Conseils & bonnes pratiques

- Toujours sauvegarder les hashs avant de les modifier
- Tester avec des wordlists personnalisées selon le contexte (profiling)
- Ne pas attaquer de systèmes sans autorisation

---

## 📚 9. Ressources

- [John the Ripper (Openwall)](https://www.openwall.com/john/)
- [Hashcat vs John - Comparatif](https://hashcat.net/forum/)
- [SecLists - Wordlists](https://github.com/danielmiessler/SecLists)

