🔑 BKCrack – Cheatsheet Cyber Sécurité

Attaque par plaintext connu contre ZipCrypto

BKCrack est un outil permettant de récupérer les clés internes ZipCrypto d’un fichier ZIP chiffré, en s'appuyant sur une portion connue du contenu.
Une fois les clés obtenues, il est possible de décrypter tous les fichiers de l’archive sans connaître le mot de passe.

1️⃣ Quand utiliser BKCrack ?

Utilise BKCrack UNIQUEMENT si ces conditions sont réunies :

✔ Le ZIP utilise ZipCrypto

Pas AES.
Vérifier avec :

zipinfo archive.zip


→ doit afficher ZipCrypto

✔ Tu connais une partie du contenu d’un fichier

Exemples courants :

Signature JPEG : FF D8 FF E0

Signature PNG : 89 50 4E 47

Signature PDF : %PDF-

Un fichier texte dont on connaît le début

Une image non compressée (Store), mais même Deflate marche si le plaintext est contigu

✔ Le fichier contient ≥ 8 octets de plaintext contigus

BKCrack nécessite 8 bytes minimum.

✔ Objectif :

Décrypter tout un ZIP

Contourner un mot de passe inconnu

Extraire des données sans bruteforce

2️⃣ Commandes essentielles de BKCrack
🔍 2.1 Lister le contenu du ZIP
bkcrack -L archive.zip


Affiche :

Index

Méthode de chiffrement

Compression

CRC

Taille
→ Permet d’identifier le fichier à utiliser pour l’attaque.

🧱 2.2 Attaque par plaintext connu (core feature)
1) Créer un fichier contenant les bytes connus

Exemple JPEG :

printf "\xFF\xD8\xFF\xE0\x00\x10\x4A\x46\x49\x46\x00\x01" > plain.jpg  

2) Lancer BKCrack pour récupérer les clés
bkcrack -C archive.zip -c fichier_chiffre -p fichier_plain


Exemple :

bkcrack -C santa-secret-memes.zip -c portrait.jpg -p plain.jpg


Si BKCrack réussit, il retourne :

Keys: <key0> <key1> <key2>

🔓 2.3 Déchiffrer un seul fichier
bkcrack -C archive.zip -c fichier_chiffre -k key0 key1 key2 -d fichier_dechiffre


Exemple :

bkcrack -C archive.zip -c portrait.jpg -k 4c0a34dd 9f68579b 9fd87f2f -d portrait_dec.zip

📦 2.4 Déchiffrer tous les fichiers de l’archive

Tu dois le faire pour chaque index du ZIP.

Forme générique :

bkcrack -C archive.zip -c <index> -k key0 key1 key2 -d output.zip

▶️ 2.5 Continuer une attaque interrompue
bkcrack --continue-attack <position>


Affiché automatiquement si l’attaque a été stoppée.

3️⃣ Workflow standard (résumé)

Analyse du ZIP

bkcrack -L archive.zip


Choisir un fichier qui contient un début connu
(ex : JPG, PNG…)

Créer le plaintext

printf "\xFF\xD8\xFF\xE0" > plain.jpg


Récupérer les clés

bkcrack -C archive.zip -c portrait.jpg -p plain.jpg


Décrypter les fichiers

bkcrack -C archive.zip -c 0 -k <keys> -d file0.zip
bkcrack -C archive.zip -c 1 -k <keys> -d file1.zip
...


Décompresser les flux Deflate
(souvent requis pour les JPG)

7z x file0.jpg -so > final_file0.jpg

4️⃣ Ce que BKCrack ne fait pas

Ne cracke pas les ZIP AES

Ne devine pas les mots de passe

Ne décompresse pas automatiquement les fichiers Deflate

Ne fonctionne pas si tu n'as aucun plaintext connu

5️⃣ Indices pour choisir un bon fichier plaintext

✔ Taille non compressée élevée
✔ Compression Store (non compressé) = idéal
✔ Fichier commençant par une signature connue
✔ CRC32 indiqué dans le ZIP
✔ Le plaintext doit être contigu
