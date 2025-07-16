# 🔍 Gobuster Cheatsheet – Directory Fuzzing

## 🚀 Commandes de base

### Directory busting basique
```bash
gobuster dir -u http://target.com -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
> Scan de base avec wordlist commune

### Avec extensions de fichiers
```bash
gobuster dir -u http://target.com -w /path/to/wordlist -x php,txt,html,js
```
> Recherche avec extensions spécifiques

### Avec codes de statut spécifiques
```bash
gobuster dir -u http://target.com -w /path/to/wordlist -s 200,204,301,302,307,401,403
```
> Filtre par codes de statut

---

## 🎯 Options avancées

### Threads et timeout
```bash
gobuster dir -u http://target.com -w /path/to/wordlist -t 50 --timeout 10s
```
> 50 threads avec timeout de 10 secondes

### Headers personnalisés
```bash
gobuster dir -u http://target.com -w /path/to/wordlist -H "User-Agent: Mozilla/5.0"
```
> Avec header User-Agent personnalisé

### Authentification
```bash
gobuster dir -u http://target.com -w /path/to/wordlist -U username -P password
```
> Avec authentification basique

### Cookies
```bash
gobuster dir -u http://target.com -w /path/to/wordlist -c "PHPSESSID=abc123"
```
> Avec cookies de session

---

## 📁 Modes de scan

### DNS subdomain enumeration
```bash
gobuster dns -d target.com -w /path/to/subdomain-wordlist
```
> Énumération de sous-domaines

### Virtual host discovery
```bash
gobuster vhost -u http://target.com -w /path/to/wordlist
```
> add --append-domain -r si spam de redirection
> Découverte de virtual hosts

### S3 bucket enumeration
```bash
gobuster s3 -w /path/to/wordlist
```
> Énumération de buckets S3

---

## 🔧 Filtres et options

### Ignorer les codes d'erreur
```bash
gobuster dir -u http://target.com -w /path/to/wordlist -b 404,403
```
> Ignore les codes 404 et 403

### Taille de réponse
```bash
gobuster dir -u http://target.com -w /path/to/wordlist --exclude-length 1234
```
> Ignore les réponses de taille 1234

### Récursion
```bash
gobuster dir -u http://target.com -w /path/to/wordlist -r
```
> Scan récursif dans les dossiers trouvés

### Profondeur de récursion
```bash
gobuster dir -u http://target.com -w /path/to/wordlist -r -d 3
```
> Récursion limitée à 3 niveaux

---

## 📊 Sortie et logging

### Sortie dans un fichier
```bash
gobuster dir -u http://target.com -w /path/to/wordlist -o results.txt
```

### Mode verbeux
```bash
gobuster dir -u http://target.com -w /path/to/wordlist -v
```

### Mode silencieux
```bash
gobuster dir -u http://target.com -w /path/to/wordlist -q
```

### Avec timestamp
```bash
gobuster dir -u http://target.com -w /path/to/wordlist --no-progress
```

---

## 🎭 Évasion et discrétion

### Délai entre requêtes
```bash
gobuster dir -u http://target.com -w /path/to/wordlist --delay 100ms
```
> Délai de 100ms entre chaque requête

### User-Agent aléatoire
```bash
gobuster dir -u http://target.com -w /path/to/wordlist --random-agent
```

### Proxy
```bash
gobuster dir -u http://target.com -w /path/to/wordlist --proxy http://127.0.0.1:8080
```
> Via proxy (ex: Burp Suite)

---

## 📚 Wordlists communes

### Seclists
```bash
/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
/usr/share/seclists/Discovery/Web-Content/common.txt
/usr/share/seclists/Discovery/Web-Content/big.txt
```

### Dirbuster
```bash
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
```

### Dirb
```bash
/usr/share/wordlists/dirb/common.txt
/usr/share/wordlists/dirb/big.txt
```

---

## 🔥 Commandes utiles

### Scan rapide
```bash
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/common.txt -x php,txt,html -t 50
```

### Scan complet
```bash
gobuster dir -u http://target.com -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,txt,html,js,asp,aspx,jsp -t 50 -s 200,204,301,302,307,401,403 -o gobuster_results.txt
```

### Scan avec authentification
```bash
gobuster dir -u http://target.com -w /path/to/wordlist -U admin -P password123 -x php,txt
```

### Scan de sous-domaines
```bash
gobuster dns -d target.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

---

## 🎯 Extensions communes par technologie

### PHP
```bash
-x php,php3,php4,php5,phtml
```

### ASP/ASP.NET
```bash
-x asp,aspx,asmx,ashx
```

### JSP
```bash
-x jsp,jspx,jsf
```

### Python
```bash
-x py,pyc
```

### Fichiers de config
```bash
-x txt,xml,conf,config,bak,backup,old
```
