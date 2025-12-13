# 🔨 John the Ripper - Cheatsheet Cyber Sécurité

Guide orienté **cracking de hash** et **password recovery** avec John the Ripper. Focus sur l'exploitation CPU et les techniques de cracking avancées.

---

## 📖 Qu'est-ce que John the Ripper ?

**John the Ripper** (John ou JtR) est un cracker de mots de passe open-source ultra-performant.

**Capacités** :
- Détection automatique de hash
- 400+ formats supportés
- Modes : dictionary, incremental, rules
- Multi-threading CPU
- Communauté active (Jumbo patch)

**Versions** :
- **John Core** : Version de base
- **John Jumbo** : Version communautaire avec 400+ formats

---

## 1️⃣ Modes de Cracking

### Modes disponibles

```bash
# Mode Single (utilise username comme base)
john --single hashes.txt

# Mode Wordlist (dictionary attack)
john --wordlist=rockyou.txt hashes.txt

# Mode Incremental (brute-force intelligent)
john --incremental hashes.txt

# Mode avec rules
john --wordlist=rockyou.txt --rules hashes.txt
```

---

## 2️⃣ Cracking Basique

### Détection automatique

```bash
# John détecte automatiquement le format
john hashes.txt

# Avec wordlist
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt

# Verbose
john --wordlist=rockyou.txt hashes.txt --verbose
```

### Format spécifique

```bash
# Lister les formats
john --list=formats

# Spécifier format
john --format=raw-md5 hashes.txt
john --format=raw-sha256 hashes.txt
john --format=NT hashes.txt

# Avec wordlist
john --format=raw-md5 --wordlist=rockyou.txt hashes.txt
```

---

## 3️⃣ Formats Courants

### Linux

```bash
# /etc/shadow
john --format=crypt shadow.txt

# Ou
john --format=sha512crypt shadow.txt

# Extraire depuis /etc/shadow
unshadow /etc/passwd /etc/shadow > unshadowed.txt
john unshadowed.txt
```

### Windows

```bash
# NTLM (hash Windows)
john --format=NT ntlm.txt

# LM (ancien format Windows)
john --format=LM lm.txt

# NetNTLMv2
john --format=netntlmv2 netntlmv2.txt
```

### Web

```bash
# MD5
john --format=raw-md5 md5.txt

# SHA-1
john --format=raw-sha1 sha1.txt

# SHA-256
john --format=raw-sha256 sha256.txt

# bcrypt (password_hash PHP)
john --format=bcrypt bcrypt.txt

# WordPress
john --format=phpass wordpress.txt
```

### Archives

```bash
# ZIP
zip2john archive.zip > zip.hash
john zip.hash

# RAR
rar2john archive.rar > rar.hash
john rar.hash

# 7z
7z2john archive.7z > 7z.hash
john 7z.hash

# Office
office2john document.docx > office.hash
john office.hash

# PDF
pdf2john document.pdf > pdf.hash
john pdf.hash
```

### SSH

```bash
# SSH private key
ssh2john id_rsa > ssh.hash
john ssh.hash
```

---

## 4️⃣ Extraction de Hash

### Scripts *2john

```bash
# ZIP
zip2john file.zip > hash.txt

# RAR
rar2john file.rar > hash.txt

# PDF
pdf2john file.pdf > hash.txt

# Office (Word, Excel, PowerPoint)
office2john document.docx > hash.txt
libreoffice2john document.odt > hash.txt

# SSH keys
ssh2john id_rsa > hash.txt

# KeePass
keepass2john database.kdbx > hash.txt

# TrueCrypt
truecrypt2john volume.tc > hash.txt

# Bitcoin wallet
bitcoin2john wallet.dat > hash.txt

# Ethereum wallet
ethereum2john keystore.json > hash.txt

# 1Password
1password2john backup.1pif > hash.txt
```

### Linux shadow

```bash
# Méthode 1 : unshadow
unshadow /etc/passwd /etc/shadow > unshadowed.txt
john unshadowed.txt

# Méthode 2 : directement
john /etc/shadow

# Méthode 3 : copier hash manuellement
# Format : username:$6$salt$hash
```

### Windows SAM

```bash
# Avec samdump2
samdump2 SYSTEM SAM > hashes.txt
john --format=NT hashes.txt

# Avec pwdump
pwdump SYSTEM SAM > hashes.txt
john --format=NT hashes.txt
```

---

## 5️⃣ Modes d'Attaque Avancés

### Single mode

```bash
# Utilise le username pour générer passwords
john --single hashes.txt

# Exemple :
# Username : admin
# Testé : admin, Admin, admin123, ADMIN, nimda, etc.
```

### Incremental mode

```bash
# Brute-force intelligent (par défaut ASCII)
john --incremental hashes.txt

# Mode spécifique
john --incremental=Alpha hashes.txt    # a-zA-Z
john --incremental=Digits hashes.txt   # 0-9
john --incremental=Alnum hashes.txt    # a-zA-Z0-9

# Custom incremental mode
john --incremental=LowerNum hashes.txt # a-z0-9
```

### Rules mode

```bash
# Règles par défaut
john --wordlist=rockyou.txt --rules hashes.txt

# Règle spécifique
john --wordlist=rockyou.txt --rules=Jumbo hashes.txt
john --wordlist=rockyou.txt --rules=KoreLogic hashes.txt

# Multiple règles
john --wordlist=rockyou.txt --rules=Single --rules=Jumbo hashes.txt
```

### Mask mode (jumbo)

```bash
# Format : ?l=lowercase ?u=uppercase ?d=digit ?s=special
john --mask='?l?l?l?l?d?d?d?d' hashes.txt

# Exemple : 4 lettres + 4 chiffres (password2023)
john --mask='?l?l?l?l?l?d?d?d?d' hashes.txt

# Avec min/max length
john --mask='?l?l?l?l?d?d?d?d' --min-length=8 --max-length=12 hashes.txt
```

---

## 6️⃣ Options de Performance

### CPU threads

```bash
# Auto-détection CPU
john hashes.txt

# Forcer nombre de threads
john --fork=4 hashes.txt

# Utiliser tous les CPU cores
john --fork=$(nproc) hashes.txt
```

### Sessions

```bash
# Nommer une session
john --session=mysession hashes.txt

# Restaurer session
john --restore=mysession

# Ou simplement
john --restore

# Lister sessions
john --list=sessions
```

### Statut

```bash
# Voir progression (pendant exécution)
# Appuyer sur une touche ou envoyer signal
john --status

# Voir ETA
john --status=mysession
```

---

## 7️⃣ Wordlists

### Wordlists intégrées

```bash
# Liste des wordlists intégrées
ls /usr/share/john/

# password.lst (wordlist par défaut)
john --wordlist=/usr/share/john/password.lst hashes.txt
```

### Wordlists externes

```bash
# RockYou
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt

# SecLists
john --wordlist=/usr/share/seclists/Passwords/Common-Credentials/10k-most-common.txt hashes.txt

# Custom wordlist
john --wordlist=custom.txt hashes.txt
```

### Générer wordlists

```bash
# Avec John
john --wordlist=base.txt --stdout --rules > expanded.txt

# Avec crunch
crunch 8 8 -t @@@@%%%% > wordlist.txt

# Avec cewl (scraping web)
cewl http://target.com -d 2 -m 5 -w wordlist.txt
```

---

## 8️⃣ Rules

### Règles intégrées

```bash
# Lister règles
john --list=rules

# Règles courantes :
# Single      : Variations username
# Wordlist    : Variations basiques
# Jumbo       : Règles étendues
# KoreLogic   : Règles professionnelles
```

### Créer règles custom

**Syntaxe** :
```
:       # Aucun changement
l       # Lowercase
u       # Uppercase
c       # Capitalize
C       # Lowercase first, uppercase rest
t       # Toggle case
$X      # Append character X
^X      # Prepend character X
```

**Exemple custom.rule** :
```ini
[List.Rules:Custom]
:
c
u
$1
$2
$3
$!
$@
c $1
c $2 $3
u $!
```

**Utilisation** :
```bash
# Ajouter règle dans john.conf
# Puis
john --wordlist=wordlist.txt --rules=Custom hashes.txt

# Ou spécifier fichier de config
john --wordlist=wordlist.txt --rules=Custom --config=john-local.conf hashes.txt
```

---

## 9️⃣ Affichage des Résultats

### Voir les mots de passe crackés

```bash
# Afficher tous les crackés
john --show hashes.txt

# Avec format spécifique
john --show --format=raw-md5 hashes.txt

# Format username:password
john --show hashes.txt

# Format complet
john --show=left hashes.txt  # Montrer non-crackés
```

### Potfile

```bash
# John sauvegarde dans ~/.john/john.pot

# Voir le potfile
cat ~/.john/john.pot

# Format : $format$hash:password

# Chercher dans potfile
grep "5f4dcc3b5aa765d61d8327deb882cf99" ~/.john/john.pot
```

---

## 🔟 Cas Pratiques

### Scénario 1 : Cracking Linux

```bash
# 1. Obtenir /etc/shadow et /etc/passwd
# Via compromission ou accès physique

# 2. Combiner avec unshadow
unshadow passwd shadow > unshadowed.txt

# 3. Cracker avec wordlist
john --wordlist=/usr/share/wordlists/rockyou.txt unshadowed.txt

# 4. Si échec, avec rules
john --wordlist=/usr/share/wordlists/rockyou.txt --rules unshadowed.txt

# 5. Si échec, incremental
john --incremental unshadowed.txt

# 6. Voir résultats
john --show unshadowed.txt
```

### Scénario 2 : Archive ZIP protégée

```bash
# 1. Extraire hash
zip2john secret.zip > zip.hash

# 2. Nettoyer (optionnel)
cat zip.hash
# Garder seulement : secret.zip:$pkzip$...

# 3. Cracker
john --wordlist=/usr/share/wordlists/rockyou.txt zip.hash

# 4. Voir password
john --show zip.hash

# 5. Unzip
unzip secret.zip
# Password : [password trouvé]
```

### Scénario 3 : Windows NTLM

```bash
# 1. Obtenir hashes (via mimikatz, secretsdump, etc.)
# Format : username:RID:LM:NTLM

# 2. Extraire NTLM seulement
cut -d: -f4 ntds.txt > ntlm.txt

# 3. Cracker
john --format=NT --wordlist=/usr/share/wordlists/rockyou.txt ntlm.txt

# 4. Avec rules
john --format=NT --wordlist=/usr/share/wordlists/rockyou.txt --rules ntlm.txt

# 5. Résultats
john --format=NT --show ntlm.txt
```

### Scénario 4 : SSH Private Key

```bash
# 1. Extraire hash de la clé
ssh2john id_rsa > ssh.hash

# 2. Cracker
john --wordlist=/usr/share/wordlists/rockyou.txt ssh.hash

# 3. Voir passphrase
john --show ssh.hash

# 4. Utiliser la clé
chmod 600 id_rsa
ssh -i id_rsa user@target
# Enter passphrase: [passphrase trouvée]
```

---

## 1️⃣1️⃣ Comparaison John vs Hashcat

### Quand utiliser John

```
✅ Pas de GPU disponible
✅ Formats spécifiques (Office, PDF, archives)
✅ Mode Single (avec usernames)
✅ Scripts *2john intégrés
✅ Détection automatique de format
✅ Règles intégrées complexes
```

### Quand utiliser Hashcat

```
✅ GPU disponible (beaucoup plus rapide)
✅ Hash simples (MD5, SHA, NTLM)
✅ Performance maximale
✅ Masks personnalisables
✅ Benchmark facile
```

### Combiné

```bash
# 1. Identifier hash avec John
john --list=formats | grep -i md5

# 2. Tester rapidement avec John
john --format=raw-md5 hashes.txt --wordlist=common.txt

# 3. Si pas de GPU, continuer avec John
john --format=raw-md5 hashes.txt --wordlist=rockyou.txt --rules

# 4. Si GPU disponible, utiliser Hashcat
hashcat -m 0 -a 0 hashes.txt rockyou.txt -r rules/best64.rule
```

---

## 1️⃣2️⃣ Scripts d'Automatisation

### Script de cracking progressif

```bash
#!/bin/bash
# john-progressive.sh

HASH_FILE=$1

echo "[+] Starting progressive attack on $HASH_FILE"

# 1. Détection auto
echo "[*] Auto-detection..."
john $HASH_FILE --wordlist=/usr/share/john/password.lst

# 2. Wordlist
echo "[*] Wordlist attack..."
john $HASH_FILE --wordlist=/usr/share/wordlists/rockyou.txt

# 3. Wordlist + rules
echo "[*] Wordlist + rules..."
john $HASH_FILE --wordlist=/usr/share/wordlists/rockyou.txt --rules

# 4. Single mode
echo "[*] Single mode..."
john --single $HASH_FILE

# 5. Incremental (si temps disponible)
echo "[*] Incremental mode..."
john --incremental $HASH_FILE

echo "[+] Attack complete!"
john --show $HASH_FILE
```

### Script multi-format

```bash
#!/bin/bash
# john-multi-format.sh

HASH=$1

echo "[+] Testing multiple formats for: $HASH"

# Formats courants
FORMATS="raw-md5 raw-sha1 raw-sha256 NT raw-sha512"

for FORMAT in $FORMATS; do
    echo "[*] Testing format: $FORMAT"
    echo "$HASH" | john --format=$FORMAT --wordlist=/usr/share/wordlists/rockyou.txt --stdin
    
    # Vérifier si cracké
    RESULT=$(echo "$HASH" | john --format=$FORMAT --show 2>/dev/null)
    if [ ! -z "$RESULT" ]; then
        echo "[+] SUCCESS with $FORMAT: $RESULT"
        break
    fi
done
```

---

## 1️⃣3️⃣ Configuration

### john.conf

```bash
# Emplacement
/etc/john/john.conf
~/.john/john.conf

# Voir config active
john --list=conf-all
```

### Modifier configuration

```ini
# ~/.john/john-local.conf

[Options]
Wordlist = /usr/share/wordlists/rockyou.txt

[Incremental:Custom]
File = $JOHN/custom.chr
MinLen = 8
MaxLen = 12
CharCount = 95

[List.Rules:MyRules]
:
c
u
$1
$2$3
```

---

## 1️⃣4️⃣ Dépannage

### Erreurs courantes

**"No password hashes loaded"**
```bash
# Vérifier format du hash
cat hashes.txt

# Spécifier format
john --format=raw-md5 hashes.txt

# Lister formats supportés
john --list=formats
```

**"No such format"**
```bash
# Installer John Jumbo (version communautaire)
git clone https://github.com/openwall/john
cd john/src
./configure && make

# Ou installer version packagée
apt install john-jumbo
```

**Session bloquée**
```bash
# Supprimer session
rm ~/.john/john.rec

# Ou forcer nouvelle session
john --session=newsession hashes.txt
```

---

## 1️⃣5️⃣ Cheatsheet Rapide

### Commandes essentielles

```bash
# Auto-detect
john hashes.txt

# Wordlist
john --wordlist=rockyou.txt hashes.txt

# Format spécifique
john --format=FORMAT hashes.txt

# Avec rules
john --wordlist=rockyou.txt --rules hashes.txt

# Incremental
john --incremental hashes.txt

# Show cracked
john --show hashes.txt

# Restore session
john --restore

# List formats
john --list=formats

# Threads
john --fork=4 hashes.txt
```

### *2john utils

```bash
zip2john file.zip > hash.txt
rar2john file.rar > hash.txt
pdf2john file.pdf > hash.txt
office2john file.docx > hash.txt
ssh2john id_rsa > hash.txt
keepass2john db.kdbx > hash.txt
```

### Formats courants

```
raw-md5          : MD5
raw-sha1         : SHA-1
raw-sha256       : SHA-256
NT               : NTLM (Windows)
crypt            : Unix crypt
sha512crypt      : Linux SHA-512
bcrypt           : bcrypt
phpass           : WordPress/phpBB
```

---

## 1️⃣6️⃣ Ressources

### Documentation
- Site officiel : https://www.openwall.com/john/
- GitHub : https://github.com/openwall/john
- Wiki : https://openwall.info/wiki/john

### Wordlists
- RockYou : https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt
- SecLists : https://github.com/danielmiessler/SecLists
- CrackStation : https://crackstation.net/crackstation-wordlist-password-cracking-dictionary.htm

### Communauté
- Mailing list : john-users@lists.openwall.com
- Forum : https://www.openwall.com/lists/john-users/

---

## 💡 Tips Pro

1. **Jumbo version** : Plus de formats supportés
2. **Unshadow** : Toujours utiliser pour Linux
3. **Rules > Incremental** : Plus efficace
4. **Fork pour CPU** : Utiliser tous les cores
5. **Potfile** : John se souvient des hash crackés
6. **Sessions** : Toujours nommer pour grandes listes
7. **Single mode** : Efficace avec usernames
8. ***2john scripts** : Essentiels pour archives
9. **Format auto-detect** : Pratique mais parfois faux
10. **Combiner avec Hashcat** : John pour extraction, Hashcat pour GPU

---

**🔨 John the Ripper est l'outil de cracking CPU le plus polyvalent. 400+ formats supportés et détection automatique !**
