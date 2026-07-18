# Hashing — Technique

> Identification, manipulation et cracking de hash cryptographiques (MD5, SHA, NTLM, bcrypt, Argon2...) en contexte pentest/CTF.

---

## Présentation

Un **hash** est une empreinte numérique de taille fixe générée à partir d'une donnée de taille quelconque. Il est :
- **Unidirectionnel** : impossible de retrouver l'entrée depuis le hash
- **Déterministe** : même entrée → même sortie
- **Sensible aux collisions** : deux entrées différentes peuvent (rarement) produire le même hash

```
"password"  → SHA256 → 5e884898da28047151d0e56f8dc6292773603d0d...
"Password"  → SHA256 → e7cf3ef04b5d2a48a4a62e3e6dd15e5c72f0e1e6...
```

### Hash / Chiffrement / Encodage

| Type | Réversible | Clé | Exemple |
|---|---|---|---|
| Hash | Non | Non | SHA256, MD5 |
| Chiffrement symétrique | Oui | Même clé | AES, DES |
| Chiffrement asymétrique | Oui | Clé pub/priv | RSA, ECC |
| Encodage | Oui | Non | Base64, Hex, UTF-8 |

> Base64 **n'est pas du chiffrement** — c'est juste une représentation de bytes en ASCII.

### Algorithmes courants

**MD5**
```
Taille      : 128 bits (32 hex)
Statut      : CASSÉ — collisions connues, ne pas utiliser
Usage actuel: Vérification d'intégrité (non-sécurisée), checksums
```
```bash
echo -n "password" | md5sum
# 5f4dcc3b5aa765d61d8327deb882cf99
```

**SHA-1**
```
Taille      : 160 bits (40 hex)
Statut      : DÉPRÉCIÉ — collisions démontrées (SHAttered 2017)
Usage actuel: Git (legacy), certificats anciens
```
```bash
echo -n "password" | sha1sum
# 5baa61e4c9b93f3f0682250b6cf8331b7ee68fd8
```

**SHA-256 / SHA-512 (SHA-2)**
```
SHA-256     : 256 bits (64 hex)
SHA-512     : 512 bits (128 hex)
Statut      : Sécurisé — recommandé pour usage général
Usage actuel: TLS, JWT, Bitcoin, stockage de mots de passe basique
```
```bash
echo -n "password" | sha256sum
# 5e884898da28047151d0e56f8dc629277360...

echo -n "password" | sha512sum
# b109f3bbbc244eb82441917ed06d618b9008dd09b3bef...
```

**SHA-3 (Keccak)**
```
Taille      : 224, 256, 384, 512 bits
Statut      : Très sécurisé — algorithme différent de SHA-2
Usage actuel: Ethereum, applications modernes
```
```bash
echo -n "password" | openssl dgst -sha3-256
```

**PBKDF2**
```
Type        : KDF (Key Derivation Function) avec itérations
Algorithme  : HMAC-SHA1 ou HMAC-SHA256 en interne
Paramètres  : sel (salt) + nombre d'itérations + taille de sortie
Statut      : Sécurisé si itérations élevées (>100 000)
Usage actuel: Mirth Connect, Django, iOS Keychain
Format hashcat (mode 10900) : sha256:iterations:sel_base64:hash_base64
```
> Exemple réel : Mirth Connect 4.0 utilise PBKDF2-HMAC-SHA256 avec 600 000 itérations.

**bcrypt**
```
Taille      : 60 caractères (format fixe avec $2a$, $2b$, $2y$)
Paramètres  : cost factor (rounds)
Statut      : Très sécurisé — conçu pour être lent
Usage actuel: PHP (password_hash), Ruby on Rails, Spring
```
```
$2a$12$LQv3c1yqBWVHxkd0LHAkCOYz6TtxMQJqhN8/LoHHXCBxWpAFuqpxO
│   │  └─────────────────── hash (53 chars)
│   └────────────────────── sel (22 chars)
└────────────────────────── $2a$ + rounds (12)
```

**Argon2**
```
Type        : KDF moderne — vainqueur du Password Hashing Competition (2015)
Variantes   : Argon2i, Argon2d, Argon2id (recommandé)
Paramètres  : mémoire, itérations, parallélisme
Statut      : Meilleur choix actuel pour mots de passe
Usage actuel: Applications modernes, libsodium
```
```
$argon2id$v=19$m=65536,t=2,p=1$sel_base64$hash_base64
```

**NTLM (Windows)**
```
Taille      : 32 hex (MD4 du mot de passe en UTF-16LE)
Statut      : Faible — pas de sel, cassable rapidement
Usage actuel: Active Directory, authentification Windows
```
Exemple de hash : `8846f7eaee8fb117ad06bdd830b7586c`

**NTLMv2 (Net-NTLMv2)**
```
Type        : Challenge-response (capture réseau)
Statut      : Plus fort que NTLM mais cassable avec responder + hashcat
Usage actuel: Authentification réseau Windows
```
```
user::DOMAIN:challenge:hash:blob
```
```bash
# Capture avec Responder
responder -I eth0
```

### Comparaison rapide

| Algo | Taille | Sel | Itérations | Vitesse GPU | Recommandé |
|---|---|---|---|---|---|
| MD5 | 32 hex | Non | Non | ~80 GH/s | Non |
| SHA-1 | 40 hex | Non | Non | ~25 GH/s | Non |
| SHA-256 | 64 hex | Non | Non | ~10 GH/s | Seul, non |
| NTLM | 32 hex | Non | Non | ~100 GH/s | Non |
| bcrypt | 60 chars | Oui | Oui | ~80 KH/s | Oui |
| PBKDF2 | variable | Oui | Oui | ~500 H/s* | Oui |
| Argon2id | variable | Oui | Oui | très lent | Oui, meilleur choix |

*avec 600 000 itérations sur RTX 2070 Super

## Détection / Identification

### Par la longueur

```
32 chars hex  → MD5 ou NTLM
40 chars hex  → SHA-1
56 chars hex  → SHA-224
64 chars hex  → SHA-256
96 chars hex  → SHA-384
128 chars hex → SHA-512
60 chars      → bcrypt ($2a$, $2b$, $2y$)
```

### Par le préfixe

```
$1$   → MD5-crypt
$2a$  → bcrypt
$2b$  → bcrypt
$5$   → SHA-256-crypt
$6$   → SHA-512-crypt
$y$   → yescrypt
sha256:N:sel:hash → PBKDF2-HMAC-SHA256
sha1:N:sel:hash   → PBKDF2-HMAC-SHA1
{SSHA} → Salted SHA-1 (LDAP)
$H$ ou $P$   → phpass (WordPress, phpBB3, Joomla)
```

### Outils d'identification

```bash
# hashid
hashid 5f4dcc3b5aa765d61d8327deb882cf99

# hash-identifier
hash-identifier
> 5f4dcc3b5aa765d61d8327deb882cf99

# haiti (plus moderne, meilleure couverture des formats récents)
haiti 5f4dcc3b5aa765d61d8327deb882cf99

# name-that-hash (nth) — suggère aussi le mode hashcat / format john correspondant
nth -t 5f4dcc3b5aa765d61d8327deb882cf99
```

## Exploitation

### Manipulation de bytes

**xxd — Dump hexadécimal**
```bash
# Dump hex d'une chaîne
echo -n "hello" | xxd
# 00000000: 6865 6c6c 6f                             hello

# Dump hex d'un fichier
xxd fichier.bin

# Afficher seulement les hex (sans ASCII)
echo -n "hello" | xxd -p
# 68656c6c6f

# Convertir hex → binaire
echo "68656c6c6f" | xxd -r -p
# hello

# Afficher N bytes à partir d'un offset
xxd -s 8 -l 16 fichier.bin
#   └─ offset  └─ longueur
```

**base64 — Encodage Base64**
```bash
# Encoder une chaîne
echo -n "hello" | base64
# aGVsbG8=

# Encoder un fichier
base64 fichier.bin

# Décoder
echo "aGVsbG8=" | base64 -d
# hello

# Décoder sans newline (utile pour les pipes)
echo "aGVsbG8=" | base64 -d | xxd

# Décoder et afficher en hex
echo "u/+LBBOUnac=" | base64 -d | xxd
# 00000000: bbff 8b14 1394 9a67                      .......g
```
> Règle Base64 : 3 bytes → 4 caractères. Un `=` ou `==` en fin = padding.

**openssl — Couteau suisse crypto**
```bash
# Générer un hash SHA256
echo -n "password" | openssl dgst -sha256

# HMAC-SHA256
echo -n "message" | openssl dgst -sha256 -hmac "clé_secrète"

# Encoder en base64
echo -n "hello" | openssl base64

# Décoder base64
echo "aGVsbG8=" | openssl base64 -d

# Générer des bytes aléatoires
openssl rand -hex 16           # 16 bytes en hex
openssl rand -base64 16        # 16 bytes en base64

# Chiffrer un fichier (AES-256-CBC)
openssl enc -aes-256-cbc -salt -in fichier.txt -out chiffre.enc -k "mot_de_passe"

# Déchiffrer
openssl enc -aes-256-cbc -d -in chiffre.enc -out dechiffre.txt -k "mot_de_passe"

# Info sur un certificat
openssl x509 -in cert.pem -text -noout

# Tester PBKDF2
openssl enc -aes-256-cbc -pbkdf2 -iter 600000 -in file.txt -out file.enc -k password
```

**Python — Manipulation de bytes**
```python
import base64
import hashlib
import binascii

# Décoder base64 → bytes bruts
raw = base64.b64decode("u/+LBBOUnadiyFBsMOoIDPLbUR0rk59kEkPU17itdrVWA/kLMt3w+w==")

# Afficher en hex
print(raw.hex())
print(binascii.hexlify(raw).decode())

# Couper les bytes (sel/hash)
salt = raw[:8]
hash_ = raw[8:]

# Ré-encoder en base64
print(base64.b64encode(salt).decode())
print(base64.b64encode(hash_).decode())

# PBKDF2 depuis Python
result = hashlib.pbkdf2_hmac(
    'sha256',           # algo
    b"password",        # mot de passe
    salt,               # sel (bytes)
    600000,             # itérations
    dklen=32            # longueur sortie
)
print(base64.b64encode(result).decode())

# Calculer des hashes courants
print(hashlib.md5(b"password").hexdigest())
print(hashlib.sha1(b"password").hexdigest())
print(hashlib.sha256(b"password").hexdigest())
```

### Cracking de hashes

Le détail des modes, options et scénarios de cracking (dictionnaire, masque, règles, GPU vs CPU, formats spéciaux) est couvert dans les fiches outil dédiées :
- **Hashcat** (GPU, `-m <mode>`) : voir `hashcat.md`
- **John the Ripper** (CPU, détection auto, scripts `*2john`) : voir `john_the_ripper.md`

Rappel de la logique générale une fois le hash identifié :
```bash
hashcat -m <mode> -a 0 hash.txt rockyou.txt      # dictionnaire, GPU
john --format=<format> --wordlist=rockyou.txt hash.txt  # dictionnaire, CPU
```

### Cas pratique — extraction PBKDF2 (Mirth Connect 4.0)

> Exemple réel tiré d'un challenge HTB.

**Identification**
```bash
# Hash dans la table PERSON_PASSWORD
u/+LBBOUnadiyFBsMOoIDPLbUR0rk59kEkPU17itdrVWA/kLMt3w+w==

# Décoder pour connaître la taille
echo "u/+LBBOUnadiyFBsMOoIDPLbUR0rk59kEkPU17itdrVWA/kLMt3w+w==" | base64 -d | wc -c
# 40 bytes → sel (8) + hash SHA256 (32)
```

**Extraction sel/hash et brute-force des paramètres** (salt_len et iterations inconnus, mot de passe déjà connu par ailleurs) :
```python
raw = base64.b64decode("u/+LBBOUnadiyFBsMOoIDPLbUR0rk59kEkPU17itdrVWA/kLMt3w+w==")
for iterations in [1000, 10000, 100000, 600000]:
    for salt_len in range(4, 33):
        salt, expected = raw[:salt_len], raw[salt_len:]
        if len(expected) >= 16 and hashlib.pbkdf2_hmac('sha256', b"MOT_DE_PASSE_CONNU", salt, iterations, dklen=len(expected)) == expected:
            print(f"iterations={iterations} salt_len={salt_len} -> sha256:{iterations}:{base64.b64encode(salt).decode()}:{base64.b64encode(expected).decode()}")
```

**Crack avec Hashcat**
```bash
hashcat -m 10900 "sha256:600000:u/+LBBOUnac=:YshQbDDqCAzy21EdK5OfZBJD1Ne4rXa1VgP5CzLd8Ps=" rockyou.txt
# Résultat: snowflake1
```

## Post-exploitation

Un hash cassé donne un mot de passe réutilisable pour authentification (SSH, RDP, SMB, applications web) — tester le password reuse sur les autres comptes/services. Pour les hash NTLM, l'attaque **pass-the-hash** permet de s'authentifier sans connaître le mot de passe en clair (mimikatz, `pth-winexe`, Impacket `psexec.py -hashes`).

## Vulnérabilités / Misconfigurations classiques

| Faiblesse | Impact |
|---|---|
| MD5/SHA1 utilisés pour des mots de passe, sans sel | Cracking quasi instantané (rainbow tables, GPU) |
| PBKDF2/bcrypt avec un nombre d'itérations trop faible | Cracking accéléré malgré un algo robuste |
| Hash exposé via API, LFI, message d'erreur ou dump de base | Fuite directe exploitable hors ligne |
| NTLM / NetNTLMv2 capturable sur le réseau | Cracking offline ou relais NTLM |
| Réutilisation du même mot de passe cassé sur plusieurs comptes | Mouvement latéral facilité |

## Outils de référence

| Outil | Usage |
|---|---|
| `hashid` / `hash-identifier` / `haiti` | Identification du type de hash |
| `xxd` / `base64` / `openssl` | Manipulation de bytes, encodage/décodage |
| Python `hashlib` / `binascii` | Calcul et manipulation de hash en script |
| Hashcat (voir `hashcat.md`) | Cracking GPU, tous formats courants |
| John the Ripper (voir `john_the_ripper.md`) | Cracking CPU, formats spéciaux (`*2john`) |
| CrackStation / Hashes.com | Lookup en ligne de hash déjà cassés |

## Ressources

- Hashcat modes : https://hashcat.net/wiki/doku.php?id=hashcat
- Hash examples : https://hashcat.net/wiki/doku.php?id=example_hashes
- CrackStation : https://crackstation.net
- Hashes.com : https://hashes.com/en/decrypt/hash
- Haiti : https://github.com/noraj/haiti
- PayloadsAllTheThings - Hashes : https://github.com/swisskyrepo/PayloadsAllTheThings
