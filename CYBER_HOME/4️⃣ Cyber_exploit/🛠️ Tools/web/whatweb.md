# 🌐 WhatWeb - Cheatsheet Cyber Sécurité

Guide orienté **fingerprinting web** et **reconnaissance de technologies** avec WhatWeb. Focus sur l'identification de CMS, frameworks, serveurs et composants web.

---

## 📖 Qu'est-ce que WhatWeb ?

**WhatWeb** est un outil de reconnaissance web qui identifie les technologies utilisées par un site web :
- CMS (WordPress, Joomla, Drupal...)
- Frameworks (Laravel, Django, Rails...)
- Serveurs web (Apache, Nginx, IIS...)
- Langages (PHP, Python, Ruby...)
- Librairies JS (jQuery, Bootstrap...)
- Plugins, versions, emails, comptes...

**Avantages** :
- Plus de 1800 plugins intégrés
- Niveaux d'agressivité configurables
- Output multi-formats (JSON, XML, CSV)
- Scan multi-cibles
- Proxy et authentification supportés

---

## ⚙️ Installation

```bash
# Kali Linux (préinstallé)
whatweb --version

# Apt
sudo apt install whatweb

# GitHub
git clone https://github.com/urbanadventurer/WhatWeb
cd WhatWeb && gem install bundler && bundle install
```

---

## 1️⃣ Scan de Base

### Commandes essentielles

```bash
# Scan simple
whatweb http://target.com

# HTTPS
whatweb https://target.com

# Avec verbose
whatweb -v http://target.com

# Très verbose (détails de chaque plugin)
whatweb -v -v http://target.com

# Sans couleurs
whatweb --no-color http://target.com
```

### Exemple de sortie

```
http://target.com [200 OK] Apache[2.4.41], Bootstrap[4.5.2],
Country[FRANCE][FR], Email[contact@target.com], HTML5,
HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)],
IP[1.2.3.4], JQuery[3.5.1], PHP[7.4.3], Script,
Title[Target - Accueil], WordPress[5.6.1], X-Powered-By[PHP/7.4.3]
```

---

## 2️⃣ Niveaux d'Agressivité

### Niveaux disponibles

```bash
# Niveau 1 (stealthy) - 1 requête par cible
whatweb -a 1 http://target.com

# Niveau 2 (unused)
whatweb -a 2 http://target.com

# Niveau 3 (agressive) - défaut, plusieurs requêtes
whatweb -a 3 http://target.com

# Niveau 4 (heavy) - bruteforce de plugins
whatweb -a 4 http://target.com
```

### Quand utiliser quel niveau

```
Niveau 1 → Pentest discret, IDS/WAF sensibles, prod sensible
Niveau 3 → Scan standard CTF, lab, recon normale
Niveau 4 → Analyse approfondie, pas de restriction de requêtes
```

---

## 3️⃣ Scan Multi-Cibles

### Depuis un fichier

```bash
# Fichier de cibles (une URL par ligne)
whatweb -i targets.txt

# Avec agressivité
whatweb -i targets.txt -a 3

# Plage IP
whatweb 192.168.1.0/24

# Plage IP sur un port spécifique
whatweb 192.168.1.0/24:8080

# Plusieurs cibles en ligne
whatweb http://target1.com http://target2.com http://target3.com
```

---

## 4️⃣ Output et Reporting

### Formats disponibles

```bash
# Log format simple
whatweb http://target.com --log-brief=brief.txt

# Log verbose
whatweb http://target.com --log-verbose=verbose.txt

# Log JSON (pour parsing)
whatweb http://target.com --log-json=output.json

# Log XML
whatweb http://target.com --log-xml=output.xml

# Log CSV (pour Excel)
whatweb http://target.com --log-csv=output.csv

# Log MongoDB
whatweb http://target.com --log-mongo=whatweb,db

# Tout à la fois
whatweb http://target.com \
    --log-json=results.json \
    --log-verbose=results.txt
```

### Parsing JSON avec jq

```bash
# Scan JSON puis parser
whatweb http://target.com --log-json=out.json
cat out.json | jq '.'

# Extraire les technologies trouvées
cat out.json | jq '.[].plugins | keys'

# Extraire les versions PHP
cat out.json | jq '.[].plugins.PHP.version'

# Extraire tous les emails
cat out.json | jq '.[].plugins.Email.string'
```

---

## 5️⃣ Threads et Performance

```bash
# Nombre de threads (défaut : 25)
whatweb -t 50 http://target.com

# Threads + multi-cibles
whatweb -i targets.txt -t 100

# Max threads (attention aux ressources)
whatweb -i targets.txt -t 200

# Délai entre requêtes (en secondes)
whatweb http://target.com --wait=0.5
```

---

## 6️⃣ Authentification

### Basic Auth

```bash
# Username et password
whatweb --user=admin --password=password http://target.com

# Format condensé
whatweb http://admin:password@target.com
```

### Cookies

```bash
# Cookie session
whatweb http://target.com --cookie="PHPSESSID=abc123xyz"

# Multiples cookies
whatweb http://target.com --cookie="session=xxx; token=yyy"
```

### Headers

```bash
# Header personnalisé
whatweb http://target.com --header="X-Custom: value"

# Authorization Bearer
whatweb http://target.com --header="Authorization: Bearer eyJhbGci..."
```

---

## 7️⃣ Proxy et SSL

```bash
# Via Burp Suite
whatweb http://target.com --proxy=http://127.0.0.1:8080

# SOCKS5
whatweb http://target.com --proxy=socks5://127.0.0.1:1080

# Ignorer les erreurs SSL
whatweb https://target.com --no-errors

# Proxy + SSL
whatweb https://target.com --proxy=http://127.0.0.1:8080
```

---

## 8️⃣ Plugins

### Gestion des plugins

```bash
# Lister tous les plugins
whatweb --list-plugins

# Lister avec détails
whatweb --list-plugins | grep -i wordpress

# Utiliser un plugin spécifique
whatweb http://target.com --plugins=WordPress

# Multiples plugins
whatweb http://target.com --plugins=WordPress,Joomla,Drupal

# Exclure un plugin
whatweb http://target.com --plugins=-WordPress

# Info sur un plugin
whatweb http://target.com --info-plugins=WordPress
```

### Plugins utiles par catégorie

```bash
# CMS
WordPress, Joomla, Drupal, DotNetNuke, Magento

# Frameworks
Laravel, Django, Rails, Symfony, CodeIgniter

# Serveurs
Apache, Nginx, IIS, Lighttpd, LiteSpeed

# Langages
PHP, Python, Ruby, ASP

# CDN / WAF
Cloudflare, Akamai, Sucuri, ModSecurity
```

---

## 9️⃣ User-Agent et Furtivité

```bash
# User-Agent personnalisé
whatweb http://target.com --user-agent="Mozilla/5.0 (X11; Linux x86_64)"

# Simuler Googlebot
whatweb http://target.com --user-agent="Googlebot/2.1"

# Niveau stealthy (1 requête)
whatweb -a 1 --user-agent="Mozilla/5.0" http://target.com

# Avec délai
whatweb -a 1 --wait=1 http://target.com
```

---

## 🔟 Cas Pratiques

### Fingerprinting WordPress

```bash
# Détection WordPress complète
whatweb -a 3 -v http://target.com --plugins=WordPress,JQuery

# Chercher version et plugins WordPress
whatweb http://target.com --log-json=wp.json
cat wp.json | jq '.[].plugins | {wp: .WordPress, plugins: ."WordPress-Plugin"}'
```

### Recon réseau local

```bash
# Scanner tout un sous-réseau
whatweb 192.168.1.0/24 -t 50 --log-csv=lan-scan.csv

# Ports non-standard
whatweb 192.168.1.0/24:8080
whatweb 192.168.1.0/24:8443

# Combiné plusieurs ports
for port in 80 443 8080 8443 8888; do
    whatweb 192.168.1.0/24:$port --log-csv=port-$port.csv
done
```

### Bug Bounty Workflow

```bash
# 1. Fingerprinting rapide
whatweb -a 1 https://target.com --log-json=fingerprint.json

# 2. Analyser les technologies
cat fingerprint.json | jq '.[].plugins | keys[]'

# 3. Scan approfondi selon les techno trouvées
whatweb -a 3 -v https://target.com --log-verbose=detailed.txt

# 4. Chercher emails et infos exposées
cat fingerprint.json | jq '.[].plugins | {email: .Email, ip: .IP}'
```

### CTF / HTB Workflow

```bash
# Scan initial complet
whatweb -a 3 -v http://10.10.10.X | tee whatweb-initial.txt

# Si WordPress détecté → lancer wpscan
whatweb http://10.10.10.X --log-json=out.json && \
    cat out.json | jq '.[].plugins.WordPress' && \
    wpscan --url http://10.10.10.X

# Chercher des versions vulnérables
whatweb -a 3 http://10.10.10.X 2>&1 | grep -iE "version|v[0-9]"
```

---

## 1️⃣1️⃣ Cheatsheet Rapide

```bash
# Scan basique
whatweb http://target.com

# Verbose
whatweb -v http://target.com

# Stealthy
whatweb -a 1 http://target.com

# Agressif
whatweb -a 4 http://target.com

# Multi-cibles
whatweb -i targets.txt -t 50

# Plage IP
whatweb 192.168.1.0/24

# Output JSON
whatweb http://target.com --log-json=out.json

# Output CSV
whatweb http://target.com --log-csv=out.csv

# Via proxy
whatweb http://target.com --proxy=http://127.0.0.1:8080

# Cookie
whatweb http://target.com --cookie="PHPSESSID=abc123"

# Plugin spécifique
whatweb http://target.com --plugins=WordPress

# Lister plugins
whatweb --list-plugins | grep -i "cms"
```

---

## 💡 Tips Pro

1. **Niveau 1** pour les environnements de prod ou sensibles aux IDS
2. **--log-json** pour parser avec jq et automatiser les décisions
3. **Combiner avec WPScan** si WordPress détecté
4. **Plages IP** pour la reconnaissance réseau interne
5. **Chercher les emails** exposés : souvent utiles pour phishing/OSINT
6. **Versions exposées** dans X-Powered-By → chercher CVE directement
7. **Niveau 4** uniquement en lab/CTF, très bruyant
8. **Croiser avec shodan/censys** sur les IPs retournées

---

**🌐 WhatWeb est l'outil de fingerprinting web de référence. Toujours lancer en premier pour orienter la suite du pentest !**
