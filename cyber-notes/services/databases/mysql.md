# 📚 MySQL Enumeration & Exploitation

## 🔍 Enumération
- `nmap -p 3306 --script=mysql-info <ip>`
- `mysql -u root -h <ip> -P 3306 -p`

## 🚪 Vulnérabilités connues
- Auth bypass (CVE...)
- SQL Injection

## 📤 Dump
- `mysqldump -u root -p --all-databases`

## 📌 Outils utiles
- `mysql-client`, `sqlmap`, `hydra`
