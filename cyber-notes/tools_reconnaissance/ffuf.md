# ⚡ ffuf - Cheatsheet Cyber Sécurité

Ce fichier regroupe les **commandes essentielles** et **astuces** pour utiliser **ffuf** (Fuzz Faster U Fool), un outil de fuzzing web très efficace pour découvrir des fichiers, répertoires, paramètres et vulnérabilités.

---

## 1️⃣ Découverte de fichiers et dossiers

ffuf -w wordlist.txt -u http://target/FUZZ  
> Fuzz sur le chemin URL pour trouver des fichiers ou dossiers cachés.

Options utiles :  
-mc 200  
> Filtrer les résultats par code HTTP (ici 200 = OK)  
-t 50  
> Nombre de threads pour accélérer le fuzzing  
-o result.json -of json  
> Exporter les résultats au format JSON

---

## 2️⃣ Fuzz des paramètres GET

ffuf -w params.txt -u http://target/page.php?FUZZ=valeur  
> Tester différents paramètres GET sur une page.

---

## 3️⃣ Fuzz des paramètres POST

ffuf -w params.txt -X POST -d "username=admin&password=FUZZ" -u http://target/login.php  
> Tester des payloads sur un champ POST.

---

## 4️⃣ Recherche de vulnérabilités communes

### Fuzz pour détecter les pages admin

ffuf -w common-admin-pages.txt -u http://target/FUZZ

### Fuzz pour trouver des points d’injection

ffuf -w injection-payloads.txt -u http://target/search?q=FUZZ

---

## 5️⃣ Utilisation avancée

### Fuzz avec plusieurs mots-clés dans l’URL

ffuf -w list1.txt -w list2.txt -u http://target/FUZZ1/FUZZ2

### Ignorer certains codes HTTP

ffuf -w wordlist.txt -u http://target/FUZZ -mc 200,301,302

---

## 6️⃣ Exemples pratiques

### Fuzz dossier avec taille de réponse différente

ffuf -w wordlist.txt -u http://target/FUZZ -fs 1234  
> Ignore les réponses avec taille 1234 octets (typiquement une page 404)

### Fuzz sous-domaines

ffuf -w subdomains.txt -u http://FUZZ.target.com

---

## 📌 Ressources utiles

- Documentation officielle : https://ffuf.dev  
- Wordlists populaires : https://github.com/danielmiessler/SecLists  
- Tutoriels et exemples : https://github.com/ffuf/ffuf/wiki  
