# ⚡ ffuf - Cheatsheet Cyber Sécurité

Guide orienté **fuzzing web** et **découverte de vulnérabilités** avec ffuf (Fuzz Faster U Fool). Focus sur l'énumération avancée et l'exploitation.

---

## 📖 Qu'est-ce que ffuf ?

**ffuf** est un fuzzer web ultra-rapide écrit en Go permettant de :
- Découvrir des fichiers et répertoires cachés
- Fuzzer des paramètres GET/POST
- Énumérer des sous-domaines
- Découvrir des virtual hosts
- Brute-forcer des credentials
- Tester des injections

**Avantages** :
- Ultra-rapide (Go)
- Multi-threading efficace
- Flexible (fuzzing multi-positions)
- Filtrage avancé

---

## 1️⃣ Directory Fuzzing

### Découverte basique

```bash
# Scan simple
ffuf -w wordlist.txt -u http://target.com/FUZZ

# Avec extensions
ffuf -w wordlist.txt -u http://target.com/FUZZ -e .php,.html,.txt,.js

# Multi-extensions
ffuf -w dirs.txt:DIRS -w exts.txt:EXTS -u http://target.com/DIRSEXTS
```

### Fuzzing récursif

```bash
# Récursion automatique
ffuf -w wordlist.txt -u http://target.com/FUZZ -recursion

# Profondeur de récursion
ffuf -w wordlist.txt -u http://target.com/FUZZ -recursion -recursion-depth 3

# Récursion avec stratégie
ffuf -w wordlist.txt -u http://target.com/FUZZ -recursion -recursion-strategy greedy
```

---

## 2️⃣ Filtrage Avancé

### Filtres par code HTTP

```bash
# Matcher codes spécifiques
ffuf -w wordlist.txt -u http://target.com/FUZZ -mc 200,301,302

# Filtrer codes (exclude)
ffuf -w wordlist.txt -u http://target.com/FUZZ -fc 404,403

# Matcher tous sauf certains
ffuf -w wordlist.txt -u http://target.com/FUZZ -mc all -fc 404
```

### Filtres par taille

```bash
# Filtrer par taille de réponse
ffuf -w wordlist.txt -u http://target.com/FUZZ -fs 1234

# Filtrer par nombre de lignes
ffuf -w wordlist.txt -u http://target.com/FUZZ -fl 42

# Filtrer par nombre de mots
ffuf -w wordlist.txt -u http://target.com/FUZZ -fw 100

# Matcher par taille
ffuf -w wordlist.txt -u http://target.com/FUZZ -ms 5000-10000
```

### Filtres par regex

```bash
# Filtrer par regex sur contenu
ffuf -w wordlist.txt -u http://target.com/FUZZ -fr "error|not found"

# Matcher par regex
ffuf -w wordlist.txt -u http://target.com/FUZZ -mr "admin|dashboard|success"
```

### Filtrage par temps de réponse

```bash
# Filtrer réponses rapides (< 100ms)
ffuf -w wordlist.txt -u http://target.com/FUZZ -ft 100

# Matcher réponses lentes (time-based blind)
ffuf -w wordlist.txt -u http://target.com/FUZZ -mt 5000
```

---

## 3️⃣ Paramètres GET/POST

### Fuzzing de paramètres GET

```bash
# Paramètre simple
ffuf -w params.txt -u http://target.com/page.php?FUZZ=test

# Valeur du paramètre
ffuf -w values.txt -u http://target.com/page.php?id=FUZZ

# Multi-paramètres
ffuf -w params.txt:PARAM -w values.txt:VAL -u "http://target.com/?PARAM=VAL"
```

### Fuzzing POST

```bash
# POST data simple
ffuf -w payloads.txt -X POST -d "username=admin&password=FUZZ" -u http://target.com/login

# JSON POST
ffuf -w payloads.txt -X POST -H "Content-Type: application/json" -d '{"user":"admin","pass":"FUZZ"}' -u http://target.com/api/login

# Fuzzing nom de paramètre POST
ffuf -w params.txt -X POST -d "FUZZ=value" -u http://target.com/submit
```

### Fuzzing de headers

```bash
# Fuzzing User-Agent
ffuf -w useragents.txt -H "User-Agent: FUZZ" -u http://target.com/

# Fuzzing custom headers
ffuf -w headers.txt -H "X-Forwarded-For: FUZZ" -u http://target.com/admin

# Fuzzing Host header (virtual host)
ffuf -w vhosts.txt -H "Host: FUZZ" -u http://target.com/
```

---

## 4️⃣ Sous-domaines et VHosts

### Énumération de sous-domaines

```bash
# DNS-based
ffuf -w subdomains.txt -u http://FUZZ.target.com

# Avec vérification HTTP
ffuf -w subdomains.txt -u http://FUZZ.target.com -mc 200

# DNS wildcard filter
ffuf -w subdomains.txt -u http://FUZZ.target.com -fw 1
```

### Virtual Hosts

```bash
# VHost discovery
ffuf -w vhosts.txt -u http://target.com -H "Host: FUZZ.target.com"

# Avec filtrage de taille
ffuf -w vhosts.txt -u http://target.com -H "Host: FUZZ.target.com" -fs 1234

# IP-based VHosts
ffuf -w vhosts.txt -u http://192.168.1.100 -H "Host: FUZZ"
```

---

## 5️⃣ Authentification

### Basic Auth

```bash
# Fuzzing username
ffuf -w users.txt -u http://target.com/admin -H "Authorization: Basic $(echo -n 'FUZZ:password' | base64)"

# Fuzzing password
ffuf -w passwords.txt -u http://target.com/admin -H "Authorization: Basic $(echo -n 'admin:FUZZ' | base64)"
```

### Cookies

```bash
# Fuzzing avec cookie
ffuf -w payloads.txt -u http://target.com/FUZZ -b "PHPSESSID=abc123xyz"

# Fuzzing de cookie
ffuf -w sessions.txt -u http://target.com/admin -b "session=FUZZ"
```

### Tokens JWT

```bash
# Fuzzing avec JWT
ffuf -w payloads.txt -u http://target.com/api/FUZZ -H "Authorization: Bearer eyJhbGc..."

# Fuzzing JWT values
ffuf -w tokens.txt -u http://target.com/api/user -H "Authorization: Bearer FUZZ"
```

---

## 6️⃣ Techniques d'Exploitation

### SQL Injection Discovery

```bash
# Fuzzing SQLi payloads
ffuf -w sqli-payloads.txt -u "http://target.com/product?id=FUZZ" -mr "error|syntax|mysql"

# Time-based SQLi
ffuf -w sqli-time.txt -u "http://target.com/product?id=FUZZ" -ft 5000

# POST SQLi
ffuf -w sqli.txt -X POST -d "username=admin&password=FUZZ" -u http://target.com/login -mr "welcome|dashboard"
```

### XSS Discovery

```bash
# XSS payloads
ffuf -w xss-payloads.txt -u "http://target.com/search?q=FUZZ" -mr "<script>|onerror|alert"

# Reflected XSS
ffuf -w xss.txt -u "http://target.com/comment?msg=FUZZ" -mr "FUZZ"

# DOM XSS
ffuf -w xss.txt -u "http://target.com/page#FUZZ"
```

### LFI/Path Traversal

```bash
# LFI testing
ffuf -w lfi-payloads.txt -u "http://target.com/page?file=FUZZ" -mr "root:|admin:"

# Path traversal
ffuf -w traversal.txt -u "http://target.com/download?file=FUZZ" -fs 0

# Double encoding
ffuf -w lfi-double-encoded.txt -u "http://target.com/?page=FUZZ"
```

### SSRF Discovery

```bash
# SSRF testing
ffuf -w ssrf-payloads.txt -u "http://target.com/fetch?url=FUZZ" -mr "localhost|127.0.0.1|internal"

# Blind SSRF (timing)
ffuf -w internal-ips.txt -u "http://target.com/proxy?url=http://FUZZ" -ft 5000
```

### Command Injection

```bash
# Command injection
ffuf -w cmd-injection.txt -u "http://target.com/ping?ip=FUZZ" -mr "root|uid=|bin"

# Blind command injection
ffuf -w commands.txt -u "http://target.com/exec?cmd=FUZZ" -ft 5000
```

---

## 7️⃣ Multi-Position Fuzzing

### Double fuzzing

```bash
# Path + extension
ffuf -w paths.txt:PATH -w exts.txt:EXT -u http://target.com/PATHEXT

# Directory + file
ffuf -w dirs.txt:DIR -w files.txt:FILE -u http://target.com/DIR/FILE

# Parameter name + value
ffuf -w params.txt:PARAM -w values.txt:VAL -u "http://target.com/?PARAM=VAL"
```

### Triple fuzzing

```bash
# Sous-domaine + path + extension
ffuf -w subs.txt:SUB -w paths.txt:PATH -w exts.txt:EXT -u http://SUB.target.com/PATHEXT

# Method + path + param
ffuf -w methods.txt:METHOD -w paths.txt:PATH -w params.txt:PARAM -X METHOD -u "http://target.com/PATH?PARAM=test"
```

---

## 8️⃣ Performance et Optimisation

### Threads et rate limiting

```bash
# Nombre de threads
ffuf -w wordlist.txt -u http://target.com/FUZZ -t 100

# Rate limiting (requêtes/seconde)
ffuf -w wordlist.txt -u http://target.com/FUZZ -rate 50

# Délai entre requêtes
ffuf -w wordlist.txt -u http://target.com/FUZZ -p 0.1-0.5
```

### Timeout

```bash
# Timeout personnalisé
ffuf -w wordlist.txt -u http://target.com/FUZZ -timeout 10

# Avec retry
ffuf -w wordlist.txt -u http://target.com/FUZZ -timeout 5 -maxtime 3600
```

---

## 9️⃣ Output et Reporting

### Formats de sortie

```bash
# JSON
ffuf -w wordlist.txt -u http://target.com/FUZZ -o results.json -of json

# HTML report
ffuf -w wordlist.txt -u http://target.com/FUZZ -o report.html -of html

# CSV
ffuf -w wordlist.txt -u http://target.com/FUZZ -o results.csv -of csv

# Markdown
ffuf -w wordlist.txt -u http://target.com/FUZZ -o report.md -of md
```

### Output détaillé

```bash
# Verbeux
ffuf -w wordlist.txt -u http://target.com/FUZZ -v

# Silencieux
ffuf -w wordlist.txt -u http://target.com/FUZZ -s

# Mode non-interactif
ffuf -w wordlist.txt -u http://target.com/FUZZ -noninteractive
```

---

## 🔟 Proxy et SSL

### Via proxy

```bash
# Proxy HTTP
ffuf -w wordlist.txt -u http://target.com/FUZZ -x http://127.0.0.1:8080

# Proxy SOCKS5
ffuf -w wordlist.txt -u http://target.com/FUZZ -x socks5://127.0.0.1:1080

# Avec authentification proxy
ffuf -w wordlist.txt -u http://target.com/FUZZ -x http://user:pass@proxy:8080
```

### SSL/TLS

```bash
# Ignorer erreurs SSL
ffuf -w wordlist.txt -u https://target.com/FUZZ -k

# Certificat client
ffuf -w wordlist.txt -u https://target.com/FUZZ -cert client.pem -key client.key
```

---

## 1️⃣1️⃣ Wordlists Recommandées

### SecLists (incontournables)

```bash
# Directories/Files
/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
/usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt
/usr/share/seclists/Discovery/Web-Content/common.txt

# API
/usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt
/usr/share/seclists/Discovery/Web-Content/swagger.txt

# Parameters
/usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt

# Subdomains
/usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt

# LFI
/usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt

# XSS
/usr/share/seclists/Fuzzing/XSS/XSS-Jhaddix.txt

# SQLi
/usr/share/seclists/Fuzzing/SQLi/Generic-SQLi.txt
```

### Wordlists custom

```bash
# Pour backup files
backup.php
backup.php.bak
backup.php~
.backup.php.swp
backup.old
backup.zip

# Pour admin panels
admin/
administrator/
admin.php
admin/login.php
wp-admin/
phpmyadmin/
```

---

## 1️⃣2️⃣ Scripts et Automatisation

### Scan complet automatisé

```bash
#!/bin/bash
# ffuf-scan.sh

TARGET=$1
WORDLIST="/usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt"

echo "[+] Scanning $TARGET"

# Directory fuzzing
ffuf -w $WORDLIST -u $TARGET/FUZZ -mc 200,301,302,403 -o dirs.json -of json -s

# Recursive on found dirs
cat dirs.json | jq -r '.results[].url' | while read url; do
    echo "[*] Recursive scan on $url"
    ffuf -w $WORDLIST -u $url/FUZZ -mc 200,301,302,403 -recursion-depth 2
done

# File extensions
ffuf -w $WORDLIST -u $TARGET/FUZZ -e .php,.html,.txt,.js,.json,.xml -mc 200 -o files.json -of json

# Virtual hosts
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u $TARGET -H "Host: FUZZ" -fs 1234

echo "[+] Scan complete!"
```

### Fuzzing parallèle

```bash
#!/bin/bash
# parallel-ffuf.sh

TARGETS="targets.txt"
WORDLIST="wordlist.txt"

cat $TARGETS | parallel -j 10 "ffuf -w $WORDLIST -u {}/FUZZ -mc 200,301,302 -o {//}-results.json -of json"
```

---

## 1️⃣3️⃣ Techniques Avancées

### Bypass de WAF

```bash
# Random User-Agent
ffuf -w wordlist.txt -u http://target.com/FUZZ -H "User-Agent: Mozilla/5.0 (compatible; Googlebot/2.1)"

# Delay aléatoire
ffuf -w wordlist.txt -u http://target.com/FUZZ -p 0.5-2.0

# Case manipulation
ffuf -w wordlist.txt -u http://target.com/FUZZ -ic

# Via proxy rotation
ffuf -w wordlist.txt -u http://target.com/FUZZ -replay-proxy http://proxy-list.txt
```

### Cache poisoning discovery

```bash
# X-Forwarded-Host fuzzing
ffuf -w payloads.txt -u http://target.com/ -H "X-Forwarded-Host: FUZZ"

# X-Original-URL
ffuf -w paths.txt -u http://target.com/ -H "X-Original-URL: FUZZ"

# X-Rewrite-URL
ffuf -w paths.txt -u http://target.com/ -H "X-Rewrite-URL: FUZZ"
```

### API fuzzing

```bash
# REST API endpoints
ffuf -w api-endpoints.txt -u http://target.com/api/v1/FUZZ -mc 200,201

# API versioning
ffuf -w versions.txt:VER -w endpoints.txt:END -u http://target.com/api/VER/END

# GraphQL
ffuf -w graphql-queries.txt -X POST -H "Content-Type: application/json" -d '{"query":"FUZZ"}' -u http://target.com/graphql
```

---

## 1️⃣4️⃣ Cas Pratiques

### Découverte de backups

```bash
# Extensions de backup
ffuf -w files.txt -u http://target.com/FUZZ -e .bak,.old,.backup,.zip,.tar.gz,.7z

# Pattern de backup
ffuf -w pages.txt:PAGE -u http://target.com/PAGE -e ~,.swp,.bak,.old

# Avec dates
ffuf -w dates.txt:DATE -w files.txt:FILE -u http://target.com/backup-DATE/FILE
```

### Enumération complète d'API

```bash
# Méthodes HTTP
ffuf -w api-endpoints.txt -u http://target.com/api/FUZZ -X GET,POST,PUT,DELETE,PATCH -mc all

# API keys fuzzing
ffuf -w api-keys.txt -u http://target.com/api/data -H "X-API-Key: FUZZ" -mc 200

# JWT fuzzing
ffuf -w jwt-secrets.txt -u http://target.com/api/user -H "Authorization: Bearer FUZZ"
```

### Bug bounty workflow

```bash
# 1. Sous-domaines
ffuf -w subdomains.txt -u http://FUZZ.target.com -mc 200 -o subs.json -of json

# 2. Pour chaque sous-domaine trouvé
cat subs.json | jq -r '.results[].url' | while read sub; do
    # 3. Directory fuzzing
    ffuf -w dirs.txt -u $sub/FUZZ -mc 200,301,302,403 -recursion
    
    # 4. Parameter fuzzing
    ffuf -w params.txt -u $sub/?FUZZ=test -mc 200
done
```

---

## 1️⃣5️⃣ Cheatsheet Rapide

### Commandes essentielles

```bash
# Directory
ffuf -w wordlist.txt -u http://target.com/FUZZ -mc 200,301,302

# Recursive
ffuf -w wordlist.txt -u http://target.com/FUZZ -recursion -recursion-depth 2

# Extensions
ffuf -w wordlist.txt -u http://target.com/FUZZ -e .php,.html,.txt

# POST
ffuf -w wordlist.txt -X POST -d "param=FUZZ" -u http://target.com/login

# Subdomains
ffuf -w subs.txt -u http://FUZZ.target.com

# VHosts
ffuf -w vhosts.txt -H "Host: FUZZ" -u http://target.com -fs 1234

# Filter size
ffuf -w wordlist.txt -u http://target.com/FUZZ -fs 1234

# Match codes
ffuf -w wordlist.txt -u http://target.com/FUZZ -mc 200,301,302,403

# Output JSON
ffuf -w wordlist.txt -u http://target.com/FUZZ -o results.json -of json

# Via proxy
ffuf -w wordlist.txt -u http://target.com/FUZZ -x http://127.0.0.1:8080
```

---

## 1️⃣6️⃣ Ressources

### Documentation
- GitHub : https://github.com/ffuf/ffuf
- Wiki : https://github.com/ffuf/ffuf/wiki
- Blog : https://codingo.io/tools/ffuf/

### Wordlists
- SecLists : https://github.com/danielmiessler/SecLists
- FuzzDB : https://github.com/fuzzdb-project/fuzzdb
- PayloadsAllTheThings : https://github.com/swisskyrepo/PayloadsAllTheThings

### Tutoriels
- HackTricks : https://book.hacktricks.xyz/pentesting-web/ffuf
- TryHackMe : ffuf room
- HackTheBox : Content Discovery

---

## 💡 Tips Pro

1. **Toujours filtrer par taille** pour éviter les faux positifs
2. **Utiliser -mc all -fc 404** plutôt que juste -mc 200
3. **Recursion automatique** pour découverte complète
4. **Multi-threading optimal** : -t 100 pour la plupart des cibles
5. **Output JSON** pour parsing automatisé
6. **Combiner avec Burp** : -x http://127.0.0.1:8080
7. **SecLists** est votre meilleur ami
8. **Automatiser** avec des scripts bash/python
9. **Rate limiting** pour éviter les bans : -rate 50
10. **Toujours vérifier manuellement** les résultats intéressants

---

**⚡ ffuf est l'outil de fuzzing web le plus rapide. Maîtrisez-le pour découvrir des endpoints cachés et des vulnérabilités !**
