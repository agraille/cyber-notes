# 🛡️ cyber-notes

> Une base de connaissances en cybersécurité — notes d'exploitation, cheatsheets, outils et schémas, organisée pour être utile sur le terrain.

[![Contributions welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg)](https://github.com/agraille/cyber-notes/pulls)

---

## 📖 À propos

Ce repo mise a jour regulierement regroupe mes notes personnelles en cybersécurité, accumulées au fil des CTFs, labs et formations. L'objectif est d'avoir une référence rapide, structurée et directement exploitable — pas un cours théorique.

Le contenu couvre l'exploitation web, réseau, Active Directory, la crypto, le reverse engineering, les outils offensifs courants, et bien plus.

---

## 🗂️ Structure

```
cyber-notes/
├── CTF/                        # Dossier perso pour CTF
│   └── SCRIPT/					# Scripts utile
│
└── CYBER_HOME/
    ├── 1️⃣  Cheatsheets/         # Références rapides (bash, réseau, python, reverse shells…)
    ├── 2️⃣  Schema/              # Schémas d'attaque (SSH pivoting, AD, web)
    ├── 3️⃣  Cyber_cours/         # Cours et supports (linux privesc, etc.)
    └── 4️⃣  Cyber_exploit/
        ├── 🌐 services/         # Exploitation par service (AD, cloud, BDD, réseau…)
        ├── 🎯 techniques/       # Techniques d'exploitation (web, binary, privesc…)
        └── 🛠️ Tools/            # Outils offensifs documentés
	├── 5️⃣ Cyber_defense/        # Cours et supports (linux privesc, etc.)
        └── 🛠️ Tools/            # Outils defensifs documentés


```

---

## 📚 Contenu

### 🌐 Services couverts

| Catégorie | Services |
|----------------------|-------------------------------|
| **Active Directory** | AD enumeration & exploitation.. |
| **Cloud**            | AWS, Azure, GCP               |
| **Bases de données** | MySQL, MSSQL, PostgreSQL, MongoDB, SQL Injection... |
| **Réseau**           | SSH, FTP, DNS, SMTP, Telnet, Pivoting... |
| **File Sharing**     | SMB, NFS... |
| **Services**         | LDAP, RDP, SNMP, WinRM... |

### 🎯 Techniques d'exploitation

| Catégorie | Contenu |
|--------------------------|--------------------------------------------------------------|
| **Web**                  | XSS, SSRF, SSTI, XXE, CSRF, SQLi, IDOR, JWT, OAuth, Deserialization, Cache Poisoning, HTTP Smuggling, Race Conditions, 2FA Bypass… |
| **Binary**               | Buffer Overflow, Format String... |
| **Privilege Escalation** | Linux, Windows, Containers, Linux Capabilities, Unix Sockets... |
| **Persistence**          | Backdoors... |
| **Supply Chain**         | Dependency Confusion, NPM attacks... |

### 🛠️ Outils documentés

`nmap` · `burpsuite` · `ffuf` · `gobuster` · `feroxbuster` · `hydra` · `hashcat` · `john` · `sqlmap` · `metasploit` · `bloodhound` · `crackmapexec` · `impacket` · `mimikatz` · `responder` · `netcat` · `tcpdump` · `wireshark` · `ghidra` · `gdb` · `radare2` · `nuclei` · `wpscan` · `linpeas/winpeas` · `pspy64` · `steghide` · `BKcrack` · `chisel`

---

## 🚀 Utilisation

Clone le repo et navigue dans les dossiers selon ton besoin :

```bash
git clone https://github.com/agraille/cyber-notes.git
cd cyber-notes
```

Les notes sont en Markdown, lisibles directement sur GitHub ou dans n'importe quel éditeur.

---

## 🤝 Contribuer

**Ce projet est ouvert à toute contribution !**

Tu as une note qui manque ? Une technique pas encore couverte ? Un outil à documenter ? **Fork et envoie une PR.**

### Comment participer

```bash
# 1. Fork le repo via GitHub

# 2. Clone ton fork
git clone https://github.com/<TON_USERNAME>/cyber-notes.git

# 3. Crée une branche pour ta contribution
git checkout -b feat/ma-nouvelle-note

# 4. Ajoute ou modifie du contenu
# Respecte la structure existante et le format Markdown

# 5. Commit et push
git add .
git commit -m "feat: ajout note sur <sujet>"
git push origin feat/ma-nouvelle-note

# 6. Ouvre une Pull Request sur GitHub 🎉
```

### Ce qui est le bienvenu

- Nouvelles notes sur des techniques ou outils non couverts
- Corrections d'erreurs ou mises à jour de commandes
- Nouveaux schémas d'attaque
- Améliorations de la documentation

### Guidelines

- Une note par fichier, nommée clairement (`nom_outil.md`, `technique.md`)
- Structure cohérente avec le reste du repo
- Exemples de commandes concrets et testés si possible
- Pas de données sensibles ou d'exploits 0-day actifs

---

## ⚠️ Disclaimer

Ce repo est à **des fins éducatives uniquement**. Toutes les techniques documentées ici doivent être utilisées exclusivement dans des environnements autorisés (labs, CTFs, bug bounty avec permission explicite). L'auteur décline toute responsabilité en cas d'utilisation malveillante.

---

## ⭐ Star le repo

Si ce projet t'a été utile, une étoile GitHub c'est toujours apprécié — ça aide à le rendre visible pour d'autres pentesters et étudiants en cybersécurité.

---

*Maintenu par [@agraille](https://github.com/agraille)*
