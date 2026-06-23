# ☢️ Nuclei - Guide Complet

Scanner de vulnérabilités rapide et personnalisable basé sur des templates.

---

## 📖 Présentation

**Nuclei** est un scanner de vulnérabilités moderne qui utilise des templates YAML pour détecter des failles de sécurité, misconfigurations, et expositions sur les applications web.

```
Avantages:
├── Ultra rapide (concurrent, Go)
├── +8000 templates communautaires
├── Personnalisable (YAML simple)
├── Faible taux de faux positifs
├── Multi-protocoles (HTTP, DNS, TCP, SSL...)
└── Intégration CI/CD facile
```

---

## 🔧 Installation

```bash
# Via Go
go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest

# Via Homebrew (macOS/Linux)
brew install nuclei

# Via apt (Kali/Debian)
apt install nuclei

# Via Docker
docker pull projectdiscovery/nuclei:latest

# Binaire précompilé
wget https://github.com/projectdiscovery/nuclei/releases/download/v3.2.0/nuclei_3.2.0_linux_amd64.zip
unzip nuclei_*.zip
chmod +x nuclei
sudo mv nuclei /usr/local/bin/

# Vérifier
nuclei -version

# Mettre à jour les templates
nuclei -ut
```

---

## 1️⃣ Utilisation Basique

### Premier scan

```bash
# Scanner une URL
nuclei -u https://example.com

# Scanner plusieurs URLs
nuclei -u https://site1.com -u https://site2.com

# Scanner depuis un fichier
nuclei -l urls.txt

# Depuis stdin
cat urls.txt | nuclei

# Avec subfinder (pipeline)
subfinder -d example.com | httpx | nuclei
```

### Output

```bash
# Sauvegarder les résultats
nuclei -l urls.txt -o results.txt

# Format JSON
nuclei -l urls.txt -json -o results.json

# Format JSONL (une ligne par résultat)
nuclei -l urls.txt -jsonl -o results.jsonl

# Format Markdown
nuclei -l urls.txt -me report/

# Sans couleurs
nuclei -l urls.txt -nc
```

---

## 2️⃣ Filtrage par Templates

### Par sévérité

```bash
# Uniquement critical et high
nuclei -l urls.txt -s critical,high

# Exclure info
nuclei -l urls.txt -es info

# Sévérités disponibles: info, low, medium, high, critical, unknown
```

### Par tags

```bash
# Tags spécifiques
nuclei -l urls.txt -tags cve,rce

# Exclure des tags
nuclei -l urls.txt -etags dos,fuzz

# Tags communs:
# cve, rce, lfi, xss, sqli, ssrf, redirect
# exposure, misconfig, default-login
# tech, panel, wordpress, joomla
# token, api, aws, azure, gcp
```

### Par type/protocole

```bash
# HTTP uniquement
nuclei -l urls.txt -type http

# DNS
nuclei -l hosts.txt -type dns

# Types: http, dns, file, network, headless, ssl, websocket
```

### Par auteur

```bash
# Templates d'un auteur
nuclei -l urls.txt -author pdteam
```

### Templates spécifiques

```bash
# Un seul template
nuclei -u https://example.com -t cves/2021/CVE-2021-44228.yaml

# Dossier de templates
nuclei -u https://example.com -t cves/2021/

# Templates personnalisés
nuclei -u https://example.com -t /path/to/custom/

# Plusieurs sources
nuclei -u https://example.com -t cves/ -t misconfigurations/
```

### Workflows

```bash
# Utiliser un workflow
nuclei -u https://example.com -w workflows/wordpress-workflow.yaml

# Lister les workflows
ls ~/.local/nuclei-templates/workflows/
```

---

## 3️⃣ Options de Scan

### Rate limiting

```bash
# Limiter les requêtes par seconde
nuclei -l urls.txt -rl 100

# Limiter les requêtes par minute
nuclei -l urls.txt -rlm 1000

# Concurrent templates
nuclei -l urls.txt -c 50

# Bulk size (hosts en parallèle)
nuclei -l urls.txt -bs 25
```

### Timeouts

```bash
# Timeout global
nuclei -l urls.txt -timeout 5

# Retries
nuclei -l urls.txt -retries 2
```

### Headers personnalisés

```bash
# Header unique
nuclei -l urls.txt -H "Authorization: Bearer TOKEN"

# Plusieurs headers
nuclei -l urls.txt -H "X-Custom: value" -H "Cookie: session=abc"

# User-Agent
nuclei -l urls.txt -H "User-Agent: Mozilla/5.0..."
```

### Proxy

```bash
# Via proxy HTTP
nuclei -l urls.txt -proxy http://127.0.0.1:8080

# SOCKS5
nuclei -l urls.txt -proxy socks5://127.0.0.1:1080

# Proxy avec auth
nuclei -l urls.txt -proxy http://user:pass@proxy:8080
```

### Interactsh (OOB testing)

```bash
# Avec serveur Interactsh (détection OOB)
nuclei -l urls.txt -interactsh-server https://interact.sh

# Désactiver Interactsh
nuclei -l urls.txt -ni
```

---

## 4️⃣ Templates Importants

### CVEs critiques

```bash
# Log4j (CVE-2021-44228)
nuclei -u https://target.com -t cves/2021/CVE-2021-44228.yaml

# Spring4Shell (CVE-2022-22965)
nuclei -u https://target.com -t cves/2022/CVE-2022-22965.yaml

# ProxyShell Exchange
nuclei -u https://target.com -t cves/2021/CVE-2021-34473.yaml

# Tous les CVEs d'une année
nuclei -l urls.txt -t cves/2023/
```

### Détection de technologies

```bash
# Détecter les technos
nuclei -l urls.txt -tags tech

# WordPress
nuclei -l urls.txt -tags wordpress

# Frameworks JS
nuclei -l urls.txt -t technologies/
```

### Default credentials

```bash
# Logins par défaut
nuclei -l urls.txt -tags default-login

# Panels d'admin
nuclei -l urls.txt -tags panel
```

### Expositions

```bash
# Fichiers exposés
nuclei -l urls.txt -tags exposure

# .git, .env, backup, etc.
nuclei -l urls.txt -t exposures/

# Tokens/secrets
nuclei -l urls.txt -t exposures/tokens/
nuclei -l urls.txt -t exposures/configs/
```

### Misconfigurations

```bash
# Toutes les misconfigs
nuclei -l urls.txt -t misconfiguration/

# CORS
nuclei -l urls.txt -t misconfiguration/cors/

# Headers de sécurité
nuclei -l urls.txt -t misconfiguration/http-missing-security-headers.yaml
```

### Cloud

```bash
# AWS
nuclei -l urls.txt -tags aws

# Azure
nuclei -l urls.txt -tags azure

# Tous les cloud
nuclei -l urls.txt -t cloud/
```

---

## 5️⃣ Créer ses Templates

### Structure basique

```yaml
id: mon-template-custom

info:
  name: Detection de ma vuln custom
  author: monpseudo
  severity: high
  description: Détecte une vulnérabilité XYZ
  tags: custom,web

http:
  - method: GET
    path:
      - "{{BaseURL}}/vulnerable-endpoint"
    
    matchers:
      - type: word
        words:
          - "vulnerable_string"
```

### Template avec plusieurs requêtes

```yaml
id: multi-step-template

info:
  name: Test multi-étapes
  author: monpseudo
  severity: medium
  tags: custom

http:
  - method: GET
    path:
      - "{{BaseURL}}/api/version"
    
    matchers:
      - type: status
        status:
          - 200

  - method: POST
    path:
      - "{{BaseURL}}/api/login"
    body: '{"user":"admin","pass":"admin"}'
    headers:
      Content-Type: application/json
    
    matchers-condition: and
    matchers:
      - type: word
        words:
          - "token"
      - type: status
        status:
          - 200
```

### Matchers avancés

```yaml
# Status code
matchers:
  - type: status
    status:
      - 200
      - 301

# Mots/strings
matchers:
  - type: word
    words:
      - "admin"
      - "password"
    condition: or

# Regex
matchers:
  - type: regex
    regex:
      - "api[_-]?key[\"']?\\s*[:=]\\s*[\"'][a-zA-Z0-9]+"

# Taille de réponse
matchers:
  - type: dsl
    dsl:
      - "len(body) > 1000"

# Combinaison (AND)
matchers-condition: and
matchers:
  - type: word
    words:
      - "success"
  - type: status
    status:
      - 200
```

### Extractors

```yaml
http:
  - method: GET
    path:
      - "{{BaseURL}}/.git/config"
    
    extractors:
      - type: regex
        name: git-url
        regex:
          - 'url\s*=\s*(https?://[^\s]+)'
        group: 1
```

### Variables et payloads

```yaml
http:
  - method: GET
    path:
      - "{{BaseURL}}/{{path}}"
    
    payloads:
      path:
        - admin
        - administrator
        - admin.php
        - login
    
    attack: sniper  # ou pitchfork, clusterbomb
```

### Template OOB (Out-of-Band)

```yaml
id: ssrf-oob

info:
  name: SSRF OOB Detection
  author: monpseudo
  severity: high

http:
  - method: GET
    path:
      - "{{BaseURL}}/fetch?url={{interactsh-url}}"
    
    matchers:
      - type: word
        part: interactsh_protocol
        words:
          - "http"
```

---

## 6️⃣ Scans Avancés

### Scan authentifié

```bash
# Avec cookie
nuclei -l urls.txt -H "Cookie: session=XXXXX"

# Avec header d'auth
nuclei -l urls.txt -H "Authorization: Bearer TOKEN"

# Fichier de config
nuclei -l urls.txt -config auth-config.yaml
```

### Headless (navigateur)

```bash
# Scan avec Chrome headless
nuclei -l urls.txt -headless

# Pour les templates qui le requièrent
nuclei -l urls.txt -t headless/
```

### Scan de code

```bash
# Analyse de fichiers locaux
nuclei -t file/ -target /path/to/code/

# Secrets dans le code
nuclei -t file/keys/ -target ./
```

### Scan réseau

```bash
# Templates réseau (TCP, etc.)
echo "192.168.1.1:22" | nuclei -t network/

# SSL/TLS
nuclei -l hosts.txt -t ssl/
```

---

## 7️⃣ Intégrations

### Pipeline complet

```bash
# Recon → Scan
subfinder -d target.com -silent | \
httpx -silent | \
nuclei -s critical,high -o vulns.txt

# Avec filtrage
subfinder -d target.com | \
httpx -silent -mc 200,301,302 | \
nuclei -tags cve -s critical,high
```

### Avec notify

```bash
# Alertes Slack/Discord/Telegram
nuclei -l urls.txt -s critical -notify

# Configurer dans ~/.config/notify/provider-config.yaml
```

### En continu (monitoring)

```bash
# Scan régulier avec nouveaux templates
nuclei -l urls.txt -ut -s critical,high

# Script cron
0 */6 * * * /usr/local/bin/nuclei -l /targets.txt -s critical -o /results/$(date +\%Y\%m\%d).txt
```

### Avec Burp

```bash
# Exporter les URLs depuis Burp (Target > Site map > Copy URLs)
# Scanner avec Nuclei
nuclei -l burp_urls.txt -s high,critical
```

---

## 8️⃣ Templates Communautaires

### Mettre à jour

```bash
# Update des templates
nuclei -ut

# Forcer le re-téléchargement
nuclei -ut -ud ~/.local/nuclei-templates
```

### Statistiques templates

```bash
# Compter les templates
nuclei -tl | wc -l

# Par sévérité
nuclei -tl -s critical | wc -l

# Par tag
nuclei -tl -tags cve | wc -l
```

### Templates customs GitHub

```bash
# Ajouter un repo de templates
nuclei -u https://target.com -t ~/custom-templates/

# Cloner des templates communautaires
git clone https://github.com/projectdiscovery/fuzzing-templates.git
nuclei -l urls.txt -t fuzzing-templates/
```

---

## 📋 Cheatsheet Rapide

```bash
# Scan basique
nuclei -u https://target.com
nuclei -l urls.txt

# Par sévérité
nuclei -l urls.txt -s critical,high

# Par tags
nuclei -l urls.txt -tags cve,rce,sqli

# Templates spécifiques
nuclei -u https://target.com -t cves/2021/CVE-2021-44228.yaml

# Output JSON
nuclei -l urls.txt -json -o results.json

# Avec proxy Burp
nuclei -l urls.txt -proxy http://127.0.0.1:8080

# Rate limit
nuclei -l urls.txt -rl 50 -c 10

# Headers custom
nuclei -l urls.txt -H "Authorization: Bearer XXX"

# Pipeline recon
subfinder -d target.com | httpx | nuclei -s critical,high

# Update templates
nuclei -ut
```

---

## 📚 Ressources

- **GitHub Officiel** : https://github.com/projectdiscovery/nuclei
- **Templates** : https://github.com/projectdiscovery/nuclei-templates
- **Documentation** : https://docs.projectdiscovery.io/tools/nuclei
- **Template Guide** : https://docs.projectdiscovery.io/templates/introduction

---

**Tags:** `#nuclei #scanner #vulnerability #templates #automation #bugbounty #recon #web`
