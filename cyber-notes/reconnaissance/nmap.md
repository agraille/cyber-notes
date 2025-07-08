# 🌐 Nmap - Scanner de ports et services

## Description
Nmap (Network Mapper) est un outil open source très puissant utilisé pour scanner des réseaux, découvrir les hôtes actifs, identifier les ports ouverts et les services associés. Il permet aussi la détection des systèmes d'exploitation, des versions des services, et peut lancer des scripts pour détecter des vulnérabilités.

Nmap est très utilisé en pentesting et en administration réseau pour cartographier un réseau, effectuer de la reconnaissance et préparer des tests d'intrusion.

---

## Commandes courantes

### Scan simple d'une IP
```bash
nmap 192.168.1.10
```

### Scan d'une plage d'adresses IP
```bash
nmap 192.168.1.0/24
```

### Scan en mode furtif (SYN scan)
```bash
nmap -sS 192.168.1.10
```

### Scan complet avec détection d'OS, versions, scripts NSE
```bash
nmap -A 192.168.1.10
```

### Scan sans ping (utile contre certains firewall)
```bash
nmap -Pn 192.168.1.10
```

### Scan avec détection des versions des services
```bash
nmap -sV 192.168.1.10
```

### Scan sur un port spécifique
```bash
nmap -p 80 192.168.1.10
```

---

## Options importantes

| Option       | Description                                               |
|--------------|-----------------------------------------------------------|
| `-sS`        | SYN scan (semi-ouvert, furtif)                            |
| `-sV`        | Détecte la version des services                           |
| `-O`         | Détecte le système d'exploitation                          |
| `-Pn`        | Ignore la phase de ping avant le scan                      |
| `--script`   | Lance un ou plusieurs scripts NSE (Nmap Scripting Engine)  |
| `-p`         | Spécifie les ports à scanner                              |
| `-A`         | Active la détection OS, version, script, traceroute      |
| `-oN/-oX`    | Sauvegarde les résultats en format normal ou XML          |

---

## Exemples avancés

### Scan avec script pour détecter les vulnérabilités HTTP
```bash
nmap --script=http-vuln* -p 80 192.168.1.10
```

### Sauvegarder le scan dans un fichier
```bash
nmap -oN resultat_nmap.txt 192.168.1.10
```

### Scan rapide des 100 ports les plus courants
```bash
nmap --top-ports 100 192.168.1.10
```

---

## Ressources et documentation

- Site officiel : [https://nmap.org](https://nmap.org)  
- Cheat sheet Nmap : [https://cheatography.com/davechild/cheat-sheets/nmap/](https://cheatography.com/davechild/cheat-sheets/nmap/)  
- Tutoriel complet : [https://nmap.org/book/man-briefoptions.html](https://nmap.org/book/man-briefoptions.html)

---

## Remarques

- Toujours vérifier la légalité avant de scanner un réseau.  
- Nmap est compatible Linux, Windows, MacOS.  
- Utiliser les scripts NSE avec précaution car certains peuvent être bruyants.

---

*Fin du fichier*
