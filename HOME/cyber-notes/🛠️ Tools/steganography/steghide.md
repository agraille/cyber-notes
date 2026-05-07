# 🛠️ Steghide - Cheatsheet Cyber Sécurité

Ce fichier regroupe les **commandes essentielles** et **techniques** pour utiliser **Steghide**, un outil de stéganographie permettant de cacher et d'extraire des données secrètes dans des images et fichiers audio.

---

## 📖 Qu'est-ce que Steghide ?

**Steghide** est un outil de stéganographie qui permet de **cacher des fichiers** dans des images (JPEG, BMP) et des fichiers audio (WAV, AU). Les données sont cachées en modifiant les bits de poids faible (LSB) des pixels/échantillons de manière cryptographiquement sécurisée.

**Formats supportés** :
- Images : JPEG, BMP
- Audio : WAV, AU

---

## 1️⃣ Installation

```bash
# Ubuntu/Debian
sudo apt install steghide

# Arch Linux
sudo pacman -S steghide

# macOS (via Homebrew)
brew install steghide

# Fedora/RHEL
sudo dnf install steghide
```

---

## 2️⃣ Cacher des données (Embed)

### Commande de base
```bash
steghide embed -cf image.jpg -ef secret.txt
```
> Cache le fichier `secret.txt` dans `image.jpg`. Un mot de passe sera demandé.

### Options avancées
```bash
# Avec mot de passe spécifié
steghide embed -cf image.jpg -ef secret.txt -p mon_password

# Sans mot de passe (ATTENTION : moins sécurisé)
steghide embed -cf image.jpg -ef secret.txt -p ""

# Avec compression personnalisée (1-9, 9 = max)
steghide embed -cf image.jpg -ef secret.txt -z 9

# Chiffrement personnalisé
steghide embed -cf image.jpg -ef secret.txt -e rijndael-128 -p password
```

### Algorithmes de chiffrement disponibles
- `rijndael-128` (AES, par défaut)
- `rijndael-192`
- `rijndael-256`
- `des`, `triple-des`
- `blowfish`, `twofish`
- `cast-128`, `cast-256`
- `saferplus`, `serpent`

---

## 3️⃣ Extraire des données (Extract)

### Commande de base
```bash
steghide extract -sf image.jpg
```
> Extrait les données cachées. Le mot de passe sera demandé.

### Options avancées
```bash
# Avec mot de passe spécifié
steghide extract -sf image.jpg -p mon_password

# Spécifier le nom du fichier de sortie
steghide extract -sf image.jpg -xf output.txt -p password

# Forcer l'écrasement si le fichier existe
steghide extract -sf image.jpg -xf output.txt -f -p password
```

---

## 4️⃣ Obtenir des informations

### Vérifier si un fichier contient des données cachées
```bash
steghide info image.jpg
```
> Affiche si des données sont présentes (nécessite le mot de passe pour les détails).

### Exemple de sortie
```
"image.jpg":
  format: jpeg
  capacity: 5.8 KB
Try to get information about embedded data ? (y/n) y
Enter passphrase: 
  embedded file "secret.txt":
    size: 1.2 KB
    encrypted: rijndael-128, cbc
    compressed: yes
```

---

## 5️⃣ Techniques d'analyse forensique

### Détecter la présence de stéganographie

#### a) Analyse statistique
```bash
# Vérifier les anomalies dans les LSB
stegdetect image.jpg

# Analyser avec stegbreak (bruteforce)
stegbreak -t j -f wordlist.txt image.jpg
```

#### b) Comparer avec l'original
```bash
# Si vous avez l'image originale
diff <(xxd original.jpg) <(xxd suspect.jpg)
```

#### c) Extraire sans mot de passe
```bash
# Tenter l'extraction sans mot de passe
steghide extract -sf image.jpg -p ""

# Bruteforce avec un script
for pass in $(cat wordlist.txt); do
    steghide extract -sf image.jpg -p "$pass" 2>/dev/null && echo "Password found: $pass" && break
done
```

---

## 6️⃣ Automatisation et scripts

### Script d'extraction batch
```bash
#!/bin/bash
# extract_all.sh - Extrait steghide de tous les JPEG

PASSWORD="magic_key"

for img in *.jpg; do
    echo "[*] Testing $img"
    steghide extract -sf "$img" -p "$PASSWORD" 2>&1 | grep -v "impossible"
done
```

### Script de bruteforce
```bash
#!/bin/bash
# steghide_bruteforce.sh

IMAGE="$1"
WORDLIST="$2"

while IFS= read -r password; do
    if steghide extract -sf "$IMAGE" -p "$password" 2>/dev/null; then
        echo "[+] Password found: $password"
        exit 0
    fi
done < "$WORDLIST"

echo "[-] Password not found"
```

**Usage** :
```bash
chmod +x steghide_bruteforce.sh
./steghide_bruteforce.sh image.jpg rockyou.txt
```

---

## 7️⃣ Bonnes pratiques CTF

### Indices à rechercher
```bash
# 1. Chercher des strings suspectes
strings image.jpg | grep -i "steghide\|password\|passphrase\|key"

# 2. Chercher des encodages base64
strings image.jpg | grep -E '^[A-Za-z0-9+/=]{20,}$'

# 3. Analyser les métadonnées EXIF
exiftool image.jpg | grep -i "comment\|description"

# 4. Vérifier la taille du fichier
ls -lh image.jpg  # Comparer avec des images similaires
```

### Mots de passe courants en CTF
```
password
secret
hidden
flag
steghide
admin
root
1234
key
magic_key
```

### Workflow typique
```bash
# 1. Vérifier les infos
steghide info image.jpg

# 2. Tenter sans mot de passe
steghide extract -sf image.jpg -p ""

# 3. Chercher des hints
strings image.jpg | grep -E "pass|key|hint"

# 4. Essayer des mots de passe courants
for pass in password secret hidden flag; do
    steghide extract -sf image.jpg -p "$pass" 2>/dev/null && break
done
```

---

## 8️⃣ Détection et contre-mesures

### Outils de détection
- **stegdetect** : Détecte plusieurs algorithmes de stéganographie
- **stegbreak** : Bruteforce pour steghide
- **zsteg** : Pour images PNG/BMP
- **binwalk** : Analyse de fichiers embarqués

### Limites de Steghide
- Ne fonctionne **pas** avec PNG (utiliser zsteg à la place)
- Détectable par analyse statistique avancée
- Vulnérable au bruteforce si mot de passe faible

---

## 9️⃣ Exemples pratiques

### Cacher un script
```bash
steghide embed -cf photo.jpg -ef exploit.sh -p "s3cr3t_k3y"
```

### Cacher plusieurs fichiers (via archive)
```bash
# Créer une archive
tar czf data.tar.gz file1.txt file2.txt file3.txt

# Cacher l'archive
steghide embed -cf image.jpg -ef data.tar.gz -p "password"

# Extraire et décompresser
steghide extract -sf image.jpg -p "password"
tar xzf data.tar.gz
```

### Chaîner avec d'autres outils
```bash
# Cacher un flag chiffré
echo "RM{flag}" | openssl enc -aes-256-cbc -salt -out flag.enc -pass pass:key
steghide embed -cf image.jpg -ef flag.enc -p "steghide_pass"

# Extraire et déchiffrer
steghide extract -sf image.jpg -p "steghide_pass"
openssl enc -d -aes-256-cbc -in flag.enc -pass pass:key
```

---

## 🔟 Dépannage

### Erreur : "steghide: impossible d'extraire des données"
- Vérifier que le fichier contient bien des données cachées
- Essayer un autre mot de passe
- Vérifier que le fichier n'est pas corrompu

### Erreur : "could not extract any data"
- Le fichier ne contient pas de données steghide
- Le format n'est pas supporté (ex: PNG)
- Le fichier a été modifié après l'embedding

### Optimisation de la capacité
```bash
# Voir la capacité disponible
steghide info image.jpg

# Si le fichier est trop gros, compresser d'abord
gzip -9 largefile.txt
steghide embed -cf image.jpg -ef largefile.txt.gz -z 9
```

---

## 📌 Ressources utiles

- Documentation officielle : http://steghide.sourceforge.net/
- Man page : `man steghide`
- Stegdetect (détection) : https://github.com/abeluck/stegdetect
- Liste de wordlists : https://github.com/danielmiessler/SecLists
- CTF writeups : https://ctftime.org/writeups?tags=steganography
- Stéganographie avancée : https://0xrick.github.io/lists/stego/

---

## 💡 Astuces de pro

1. **Toujours vérifier `strings`** avant d'utiliser steghide
2. **Tester sans mot de passe** en premier (`-p ""`)
3. **Combiner avec binwalk** pour détecter d'autres fichiers cachés
4. **Utiliser exiftool** pour chercher des hints dans les métadonnées
5. **En CTF, le hint est souvent dans le nom du fichier ou le challenge**
6. **Garder une wordlist de mots de passe courants pour steghide**
7. **Comparer les tailles de fichiers** pour détecter des anomalies
