# 🔍 Feroxbuster - Cheatsheet Cyber Sécurité

Guide orienté **énumération web** et **reconnaissance** avec Feroxbuster. Focus sur la découverte de ressources cachées, la récursion automatique et le fuzzing avancé.

---

## 📖 Qu'est-ce que Feroxbuster ?

**Feroxbuster** est un outil de bruteforce de répertoires et fichiers web écrit en Rust permettant de :
- Bruteforce de directories et fichiers
- Récursion automatique et intelligente
- Filtrage avancé des réponses
- Scan multi-cibles simultané
- Extraction automatique de liens dans les réponses

**Avantages** :
- Extrêmement rapide (Rust)
- Récursion automatique (killer feature)
- Filtrage très granulaire (taille, mots, lignes, regex)
- Auto-calibration (auto-tune, auto-bail)
- Support HTTP/1.1 et HTTP/2
- Configuration persistante via fichier

---

## 1️⃣ Scan de Base

### Commandes essentielles

```bash
# Scan simple
feroxbuster -u http://target.com

# Avec wordlist personnalisée
feroxbuster -u http://target.com -w /usr/share/wordlists/dirb/common.txt

# Avec extensions
feroxbuster -u http://target.com -x php,html,txt,js

# Multiples extensions
feroxbuster -u http://target.com -x php,php3,php5,phtml,html,htm,js,json,xml,txt
```

### Options de base

```bash
# Threads (défaut : 50)
feroxbuster -u http://target.com -t 100

# Profondeur de récursion (défaut : 4)
feroxbuster -u http://target.com -d 3

# Timeout (en secondes)
feroxbuster -u http://target.com --timeout 10

# Désactiver la récursion
feroxbuster -u http://target.com -n

# User-Agent custom
feroxbuster -u http://target.com -a "Mozilla/5.0 (X11; Linux x86_64)"

# Random User-Agent
feroxbuster -u http://target.com --random-agent
```

---

## 2️⃣ Récursion Automatique (Killer Feature)

### Récursion intelligente

```bash
# Récursion automatique (activée par défaut)
feroxbuster -u http://target.com -w wordlist.txt

# Limiter la profondeur
feroxbuster -u http://target.com -d 2

# Désactiver la récursion
feroxbuster -u http://target.com -n

# Récursion uniquement sur 301/302
feroxbuster -u http://target.com -d 3 -s 200,301,302
```

### Force Recursion

```bash
# Forcer la récursion même sans trailing slash
feroxbuster -u http://target.com --force-recursion

# Récursion avec extensions
feroxbuster -u http://target.com -x php,html -d 3 --force-recursion
```

---

## 3️⃣ Filtrage des Résultats

### Par code de statut

```bash
# Codes à afficher
feroxbuster -u http://target.com -s 200,204,301,302,307,401,403

# Codes à ignorer (blacklist)
feroxbuster -u http://target.com -C 404,403

# Status 200 seulement
feroxbuster -u http://target.com -s 200
```

### Par taille de réponse

```bash
# Filtrer par taille (bytes)
feroxbuster -u http://target.com --filter-size 1234

# Filtrer plusieurs tailles
feroxbuster -u http://target.com --filter-size 0,1234,5678

# Similaire à (taille similaire)
feroxbuster -u http://target.com --filter-similar-to http://target.com/404
```

### Par nombre de mots / lignes

```bash
# Filtrer par nombre de mots
feroxbuster -u http://target.com --filter-words 25

# Filtrer par nombre de lignes
feroxbuster -u http://target.com --filter-lines 10

# Combiné
feroxbuster -u http://target.com --filter-words 25 --filter-lines 10
```

### Par regex

```bash
# Filtrer par regex (contenu réponse)
feroxbuster -u http://target.com --filter-regex "not found|error|forbidden"

# Filtrer les pages d'erreur custom
feroxbuster -u http://target.com --filter-regex "Page introuvable|404 Error"
```

---

## 4️⃣ Auto-Tune et Auto-Bail

### Auto-calibration

```bash
# Auto-tune (ajustement automatique des threads)
feroxbuster -u http://target.com --auto-tune

# Auto-bail (arrêt si trop d'erreurs)
feroxbuster -u http://target.com --auto-bail

# Combiné (recommandé)
feroxbuster -u http://target.com --auto-tune --auto-bail
```

### Rate limiting

```bash
# Requêtes par seconde max
feroxbuster -u http://target.com --rate-limit 100

# Délai entre requêtes (en milliseconds)
feroxbuster -u http://target.com --rate-limit 10
```

---

## 5️⃣ Authentification

### Basic Auth

```bash
# Username et password
feroxbuster -u http://target.com --username admin --password password
```

### Cookies

```bash
# Cookie simple
feroxbuster -u http://target.com -b "PHPSESSID=abc123xyz"

# Multiples cookies
feroxbuster -u http://target.com -b "session=xxx; token=yyy"
```

### Headers custom

```bash
# Header personnalisé
feroxbuster -u http://target.com -H "X-Custom-Header: value"

# Authorization
feroxbuster -u http://target.com -H "Authorization: Bearer eyJhbGci..."

# Multiples headers
feroxbuster -u http://target.com -H "Authorization: Bearer token" -H "X-API-Key: key123"
```

---

## 6️⃣ Proxy et SSL

### Via proxy

```bash
# HTTP Proxy (Burp Suite)
feroxbuster -u http://target.com --proxy http://127.0.0.1:8080

# SOCKS5 Proxy
feroxbuster -u http://target.com --proxy socks5://127.0.0.1:1080

# Avec authentification proxy
feroxbuster -u http://target.com --proxy http://user:pass@proxy:8080
```

### SSL/TLS

```bash
# Ignorer les erreurs de certificat
feroxbuster -u https://target.com -k

# Client certificate
feroxbuster -u https://target.com --client-cert cert.pem --client-key key.pem

# Proxy + SSL ignore
feroxbuster -u https://target.com --proxy http://127.0.0.1:8080 -k
```

---

## 7️⃣ Scan Multi-Cibles

### Plusieurs URLs

```bash
# Fichier de cibles
feroxbuster --stdin < targets.txt

# Via stdin avec pipe
cat targets.txt | feroxbuster --stdin -w wordlist.txt

# Multiples URLs en ligne
feroxbuster -u http://target1.com -u http://target2.com -w wordlist.txt
```

---

## 8️⃣ Extraction de Liens

### Crawling hybride

```bash
# Extraire les liens trouvés dans les réponses
feroxbuster -u http://target.com -x html --extract-links

# Extraction + récursion
feroxbuster -u http://target.com --extract-links -d 3

# Extraction sans extensions
feroxbuster -u http://target.com --extract-links -n
```

---

## 9️⃣ Output et Reporting

### Formats de sortie

```bash
# Output dans fichier texte
feroxbuster -u http://target.com -o results.txt

# Output JSON (pour parsing)
feroxbuster -u http://target.com --json -o results.json

# Quiet mode (résultats seulement)
feroxbuster -u http://target.com -q

# Silent mode (aucun output en dehors des résultats)
feroxbuster -u http://target.com --silent

# Verbose mode
feroxbuster -u http://target.com -v
```

### Contrôle du flux

```bash
# Sans couleurs
feroxbuster -u http://target.com --no-color

# Pas de barre de progression
feroxbuster -u http://target.com --no-state

# Sauvegarde d'état (resume)
feroxbuster -u http://target.com --resume-from ferox-http_target_com.state
```

---

## 🔟 Fichier de Configuration

### ferox-config.toml

```toml
# ~/.config/feroxbuster/ferox-config.toml

# Wordlist par défaut
wordlist = "/usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt"

# Threads
threads = 50

# Profondeur de récursion
depth = 4

# Extensions par défaut
extensions = ["php", "html", "txt", "js"]

# Codes à filtrer
filter_status = [404]

# Timeout
timeout = 10

# User-Agent
user_agent = "feroxbuster/2.x"

# Output format
json = false

# Auto-tune
auto_tune = true
```

### Utilisation du config

```bash
# Utilise automatiquement ~/.config/feroxbuster/ferox-config.toml
feroxbuster -u http://target.com

# Config personnalisée
feroxbuster -u http://target.com --config /path/to/custom-config.toml

# Générer un config exemple
feroxbuster --config /dev/null --save-state
```

---

## 1️⃣1️⃣ Wordlists Recommandées

### SecLists

```bash
# Directories
/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
/usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt
/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
/usr/share/seclists/Discovery/Web-Content/common.txt

# Files
/usr/share/seclists/Discovery/Web-Content/raft-large-files.txt
/usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt

# API
/usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt
/usr/share/seclists/Discovery/Web-Content/graphql.txt

# Backups & configs
/usr/share/seclists/Discovery/Web-Content/CommonBackupExtensions.txt
/usr/share/seclists/Discovery/Web-Content/CommonBackupFilenames.txt
```

### Dirb / Dirbuster

```bash
# Dirb
/usr/share/wordlists/dirb/common.txt
/usr/share/wordlists/dirb/big.txt

# Dirbuster
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
```

---

## 1️⃣2️⃣ Extensions par Technologie

### PHP

```bash
feroxbuster -u http://target.com -x php,php3,php4,php5,php7,phtml,phar
```

### ASP/ASP.NET

```bash
feroxbuster -u http://target.com -x asp,aspx,asmx,ashx,axd,config
```

### JSP/Java

```bash
feroxbuster -u http://target.com -x jsp,jspx,jsf,jhtml,do,action
```

### Python / Django / Flask

```bash
feroxbuster -u http://target.com -x py,pyc,pyo
```

### Config & Backup

```bash
feroxbuster -u http://target.com -x txt,xml,conf,config,bak,backup,old,save,swp,env,log
```

### Archives

```bash
feroxbuster -u http://target.com -x zip,tar,tar.gz,tgz,7z,rar,bz2,gz
```

---

## 1️⃣3️⃣ Scripts et Automatisation

### Scan complet automatisé

```bash
#!/bin/bash
# feroxbuster-full-scan.sh

TARGET=$1
THREADS=50
OUTPUT_DIR="./ferox-$(date +%Y%m%d_%H%M%S)"

mkdir -p $OUTPUT_DIR
echo "[+] Starting enumeration on $TARGET"
echo "[+] Output dir: $OUTPUT_DIR"

# Directory scan avec récursion
echo "[*] Directory enumeration (recursive)..."
feroxbuster -u $TARGET \
    -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt \
    -t $THREADS -d 3 \
    -s 200,301,302,403 \
    -o $OUTPUT_DIR/dirs.txt

# File scan avec extensions
echo "[*] File enumeration..."
feroxbuster -u $TARGET \
    -w /usr/share/seclists/Discovery/Web-Content/raft-large-files.txt \
    -t $THREADS -n \
    -x php,html,txt,js,xml,json,env \
    -s 200,301,302 \
    -o $OUTPUT_DIR/files.txt

# Backups
echo "[*] Backup files..."
feroxbuster -u $TARGET \
    -w /usr/share/seclists/Discovery/Web-Content/CommonBackupFilenames.txt \
    -t $THREADS -n \
    -x bak,old,backup,~,swp,save \
    -o $OUTPUT_DIR/backups.txt

# API endpoints
echo "[*] API enumeration..."
feroxbuster -u $TARGET/api \
    -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt \
    -t $THREADS \
    -o $OUTPUT_DIR/api.txt

echo "[+] Enumeration complete! Results in $OUTPUT_DIR"
```

### Scan multi-cibles

```bash
#!/bin/bash
# multi-target-ferox.sh

TARGETS="targets.txt"
WORDLIST="/usr/share/wordlists/dirb/common.txt"

while IFS= read -r target; do
    echo "[+] Scanning $target"
    clean=$(echo $target | sed 's/https\?:\/\///g' | tr '/' '_')
    feroxbuster -u "$target" \
        -w $WORDLIST \
        -t 50 -d 2 \
        -x php,html,txt \
        --auto-bail \
        -o "${clean}.txt"
done < "$TARGETS"
```

---

## 1️⃣4️⃣ Techniques Avancées

### Bypass WAF / IDS

```bash
# User-Agent aléatoire
feroxbuster -u http://target.com --random-agent

# Rate limiting pour éviter les blocks
feroxbuster -u http://target.com --rate-limit 20

# Via proxy pour rotation IP
feroxbuster -u http://target.com --proxy http://rotating-proxy:8080

# Auto-tune pour s'adapter
feroxbuster -u http://target.com --auto-tune --auto-bail

# Délai entre requêtes
feroxbuster -u http://target.com --rate-limit 5
```

### Découverte de backups

```bash
# Extensions de backup classiques
feroxbuster -u http://target.com -w pages.txt -x bak,old,backup,save,swp,~

# Archives
feroxbuster -u http://target.com -w wordlist.txt -x zip,tar.gz,7z,rar

# Fichiers de configuration exposés
feroxbuster -u http://target.com -w wordlist.txt -x env,config,conf,xml,yaml,yml,json
```

### API Discovery

```bash
# REST API
feroxbuster -u http://target.com/api/v1 -w api-endpoints.txt

# Toutes versions d'API
for v in v1 v2 v3 v4; do
    feroxbuster -u http://target.com/api/$v \
        -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt \
        -o api-$v.txt
done

# GraphQL discovery
feroxbuster -u http://target.com -w wordlist.txt -x graphql
```

### Filter similar (pages 404 custom)

```bash
# Calibrer sur la page 404 du site
feroxbuster -u http://target.com -w wordlist.txt \
    --filter-similar-to http://target.com/cette-page-nexiste-pas-12345

# Multi-filter
feroxbuster -u http://target.com -w wordlist.txt \
    --filter-similar-to http://target.com/fakepage1 \
    --filter-similar-to http://target.com/fakepage2
```

---

## 1️⃣5️⃣ Cas Pratiques

### WordPress

```bash
# WordPress structure complète
feroxbuster -u http://target.com -w wordlist.txt -x php -d 3

# Plugins
feroxbuster -u http://target.com/wp-content/plugins \
    -w /usr/share/seclists/Discovery/Web-Content/CMS/wordpress-plugins.txt -n

# Themes
feroxbuster -u http://target.com/wp-content/themes \
    -w /usr/share/seclists/Discovery/Web-Content/CMS/wordpress-themes.txt -n

# Uploads (fichiers uploadés)
feroxbuster -u http://target.com/wp-content/uploads \
    -w years.txt -x jpg,png,pdf,docx,zip
```

### Admin Panels

```bash
# Panels d'administration
feroxbuster -u http://target.com \
    -w /usr/share/seclists/Discovery/Web-Content/admin-panels.txt -n

# Chemins communs
feroxbuster -u http://target.com -w wordlist.txt \
    --filter-regex "login|admin|dashboard|panel|console|manage"
```

### Bug Bounty Workflow

```bash
# 1. Scan initial rapide
feroxbuster -u https://target.com \
    -w /usr/share/seclists/Discovery/Web-Content/common.txt \
    -t 100 -d 1 -s 200,301,302,403

# 2. Scan profond sur chemins intéressants
feroxbuster -u https://target.com/api \
    -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt \
    -d 3 -x json --extract-links

# 3. Extraction de liens + récursion
feroxbuster -u https://target.com \
    -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt \
    --extract-links -d 4 --auto-tune --auto-bail \
    -o scope.txt
```

### Scan HTTPS avec Burp

```bash
# Intercepter dans Burp Suite
feroxbuster -u https://target.com \
    -w wordlist.txt \
    --proxy http://127.0.0.1:8080 \
    -k
```

---

## 1️⃣6️⃣ Commandes Essentielles

### Scan rapide

```bash
feroxbuster -u http://target.com -w /usr/share/wordlists/dirb/common.txt -t 50 -x php,html,txt -d 2
```

### Scan complet avec récursion

```bash
feroxbuster -u http://target.com \
    -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt \
    -t 50 -d 4 \
    -x php,html,txt,js,xml,json,zip,bak \
    -s 200,301,302,401,403 \
    --auto-tune --auto-bail \
    -o full-scan.txt
```

### Scan furtif (anti-WAF)

```bash
feroxbuster -u http://target.com \
    -w wordlist.txt \
    --random-agent \
    --rate-limit 20 \
    --auto-tune \
    -t 10 \
    -q
```

### Extraction de liens

```bash
feroxbuster -u http://target.com -w wordlist.txt --extract-links -d 3 -o crawl.txt
```

### Via Burp Suite

```bash
feroxbuster -u http://target.com -w wordlist.txt --proxy http://127.0.0.1:8080 -k
```

### JSON output (pour parsing)

```bash
feroxbuster -u http://target.com -w wordlist.txt --json -o results.json
```

---

## 1️⃣7️⃣ Comparaison avec d'autres outils

### Feroxbuster vs Gobuster

```
Feroxbuster :
+ Récursion automatique et intelligente
+ Filtrage ultra-granulaire (taille, mots, lignes, regex, similar)
+ Auto-tune / Auto-bail
+ Extraction de liens
+ Fichier de configuration persistant
+ Resume après interruption
- Interface légèrement plus complexe
- Consomme plus de mémoire

Gobuster :
+ Simple et direct
+ Stable et très connu
+ Modes DNS, VHost, S3 intégrés
- Pas de récursion automatique
- Filtrage moins avancé
- Pas d'auto-calibration
```

### Feroxbuster vs ffuf

```
Feroxbuster :
+ Récursion automatique
+ Auto-tune / Auto-bail
+ Meilleur pour l'énumération récursive
- Moins flexible pour le fuzzing multi-paramètre

ffuf :
+ Fuzzing multi-position (FUZZ dans URL, headers, body)
+ Filtrage très avancé
+ Plus flexible pour les cas custom
- Pas de récursion automatique
- Courbe d'apprentissage plus élevée
```

### Feroxbuster vs Dirsearch

```
Feroxbuster :
+ Plus rapide (Rust vs Python)
+ Récursion automatique
+ Meilleure gestion des threads

Dirsearch :
+ Nombreuses options built-in
+ Bon pour les débutants
- Plus lent
- Récursion manuelle
```

---

## 1️⃣8️⃣ Cheatsheet Rapide

```bash
# Scan basique
feroxbuster -u http://target.com

# Avec wordlist + extensions
feroxbuster -u http://target.com -w wordlist.txt -x php,html,txt

# Sans récursion
feroxbuster -u http://target.com -n

# Profondeur de récursion
feroxbuster -u http://target.com -d 2

# Threads
feroxbuster -u http://target.com -t 100

# Status codes
feroxbuster -u http://target.com -s 200,301,302,403

# Filtrer codes
feroxbuster -u http://target.com -C 404,403

# Filtrer taille
feroxbuster -u http://target.com --filter-size 1234

# Filtrer mots
feroxbuster -u http://target.com --filter-words 25

# Filtrer regex
feroxbuster -u http://target.com --filter-regex "not found"

# Filtrer similar (404 custom)
feroxbuster -u http://target.com --filter-similar-to http://target.com/fakepage

# Output fichier
feroxbuster -u http://target.com -o results.txt

# Output JSON
feroxbuster -u http://target.com --json -o results.json

# Extraction de liens
feroxbuster -u http://target.com --extract-links

# Auto-tune + Auto-bail
feroxbuster -u http://target.com --auto-tune --auto-bail

# Proxy (Burp)
feroxbuster -u http://target.com --proxy http://127.0.0.1:8080

# SSL ignore
feroxbuster -u https://target.com -k

# Cookie
feroxbuster -u http://target.com -b "PHPSESSID=abc123"

# Header custom
feroxbuster -u http://target.com -H "Authorization: Bearer token"

# Random User-Agent
feroxbuster -u http://target.com --random-agent

# Rate limit
feroxbuster -u http://target.com --rate-limit 50

# Multi-cibles (stdin)
cat targets.txt | feroxbuster --stdin -w wordlist.txt

# Resume
feroxbuster --resume-from ferox-http_target_com.state
```

---

## 1️⃣9️⃣ Ressources

### Documentation
- GitHub : https://github.com/epi052/feroxbuster
- Wiki : https://epi052.github.io/feroxbuster-docs/

### Wordlists
- SecLists : https://github.com/danielmiessler/SecLists
- Assetnote : https://wordlists.assetnote.io/

### Tutoriels
- HackTricks : https://book.hacktricks.xyz/pentesting-web/feroxbuster
- TryHackMe : Content Discovery room
- HackTheBox : Enumeration challenges

---

## 💡 Tips Pro

1. **--filter-similar-to** pour gérer les pages 404 custom sans faux positifs
2. **--extract-links** pour un crawling hybride bruteforce + spider
3. **--auto-tune --auto-bail** toujours activer en Bug Bounty
4. **-d 2 ou 3** suffisent dans 90% des cas, évite l'explosion de requêtes
5. **--json** pour parser les résultats avec jq ou en script
6. **Config toml** pour ne pas réécrire les mêmes options à chaque fois
7. **--resume-from** si le scan plante en milieu de route
8. **--rate-limit** indispensable pour les cibles sensibles ou protégées
9. **-C 404** plutôt que **-s** pour une approche permissive et filtrer ensuite
10. **Combiner avec Burp** via --proxy pour analyser le trafic manuellement

---

**🦀 Feroxbuster est l'outil d'énumération le plus puissant pour la récursion automatique. Indispensable pour une reconnaissance web complète et efficace !**
