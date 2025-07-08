# 🎯 Nmap Cheatsheet – Reconnaissance

## 🔍 Scans de base

### Scan rapide
```bash
nmap -T4 -F <target>
```
> Scan rapide des 100 ports les plus communs

### Scan complet
```bash
nmap -p- <target>
```
> Scan de tous les ports (1-65535)

### Scan avec version des services
```bash
nmap -sV <target>
```
> Détecte les versions des services

### Scan avec détection OS
```bash
nmap -O <target>
```
> Détecte le système d'exploitation

---

## 🚀 Scans avancés

### Scan agressif complet
```bash
nmap -A -T4 <target>
```
> Combine -sV, -O, -sC (scripts par défaut)

### Scan UDP
```bash
nmap -sU <target>
```
> Scan des ports UDP (plus lent)

### Scan SYN Stealth
```bash
nmap -sS <target>
```
> Scan furtif (par défaut si root)

### Scan de réseau
```bash
nmap -sn 192.168.1.0/24
```
> Découverte d'hôtes (ping scan)

---

## 📜 Scripts NSE

### Scripts par défaut
```bash
nmap -sC <target>
```
> Exécute les scripts safe par défaut

### Scripts spécifiques
```bash
nmap --script vuln <target>
```
> Scripts de détection de vulnérabilités

### Scripts HTTP
```bash
nmap -p 80,443 --script http-enum <target>
```
> Énumération de directories web

### Scripts SMB
```bash
nmap -p 445 --script smb-enum-shares <target>
```
> Énumération des partages SMB

---

## ⚡ Optimisation et timing

### Templates de timing
```bash
nmap -T0  # Paranoïaque (très lent)
nmap -T1  # Furtif
nmap -T2  # Poli
nmap -T3  # Normal (défaut)
nmap -T4  # Agressif
nmap -T5  # Insane (très rapide)
```

### Contrôle de débit
```bash
nmap --max-rate 1000 <target>
```
> Limite à 1000 paquets/seconde

---

## 🎭 Évasion et discrétion

### Fragmentation
```bash
nmap -f <target>
```
> Fragmente les paquets

### Adresse source aléatoire
```bash
nmap -D RND:10 <target>
```
> Utilise 10 adresses sources aléatoires

### Spoofing MAC
```bash
nmap --spoof-mac 0 <target>
```
> Génère une adresse MAC aléatoire

---

## 📊 Formats de sortie

### Sortie normale
```bash
nmap -oN scan.txt <target>
```

### Sortie XML
```bash
nmap -oX scan.xml <target>
```

### Sortie greppable
```bash
nmap -oG scan.gnmap <target>
```

### Toutes les sorties
```bash
nmap -oA scan <target>
```

---

## 🔥 Commandes utiles

### Scan rapide réseau avec services
```bash
nmap -sS -sV -T4 -F 192.168.1.0/24
```

### Scan complet d'un hôte
```bash
nmap -sS -sV -sC -O -p- -T4 <target>
```

### Scan web avec scripts
```bash
nmap -p 80,443 -sV -sC --script http-* <target>
```

### Scan vulnérabilités
```bash
nmap --script vuln -sV <target>
```

---

## 🎯 Cibles spécifiques

### Ports spécifiques
```bash
nmap -p 22,80,443,3389 <target>
```

### Plage de ports
```bash
nmap -p 1-1000 <target>
```

### Exclure des ports
```bash
nmap -p- --exclude-ports 22,80 <target>
```

---

## 🔍 Post-scan

### Recherche dans les résultats
```bash
grep -E "(open|filtered)" scan.gnmap
```

### Extraction des ports ouverts
```bash
grep -oP '\d+/open' scan.gnmap | cut -d'/' -f1 | sort -n
```
