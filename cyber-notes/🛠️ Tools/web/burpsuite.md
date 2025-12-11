# 🛠 Burp Suite - Cheatsheet Cyber Sécurité

Ce fichier regroupe les **fonctionnalités clés** et **astuces essentielles** pour utiliser **Burp Suite** dans un contexte de tests d’intrusion web, notamment pour détecter et exploiter des vulnérabilités comme XSS, SQLi, RFI, etc.

---

## 1️⃣ Configuration initiale

- Configurer le proxy de Burp sur l’adresse locale (127.0.0.1) et port (8080 par défaut)  
> Permet d’intercepter et modifier les requêtes HTTP/HTTPS envoyées par le navigateur.

- Installer le certificat CA Burp dans le navigateur  
> Nécessaire pour intercepter les requêtes HTTPS sans erreurs.

---

## 2️⃣ Interception & Analyse des requêtes

- Activer l’onglet **Proxy > Intercept** pour capturer les requêtes en temps réel  
> Modifier à la volée les requêtes avant qu’elles n’atteignent le serveur.

- Utiliser **HTTP history** pour voir toutes les requêtes passées  
> Analyse rétroactive des échanges HTTP/HTTPS.

---

## 3️⃣ Scanner automatique (Burp Scanner - version Pro)

- Lancer un scan de sécurité automatique sur une URL cible  
> Identifie rapidement un grand nombre de vulnérabilités courantes.

- Examiner les résultats dans l’onglet **Scanner > Issues**  
> Voir les vulnérabilités détectées avec détails et recommandations.

---

## 4️⃣ Repeater - Tests manuels

- Envoyer une requête interceptée dans l’onglet **Repeater**  
> Tester manuellement des modifications sur les paramètres, headers, cookies, etc.

- Modifier et renvoyer la requête pour observer les réponses du serveur  
> Permet d’affiner l’exploitation de vulnérabilités (ex: injection, XSS).

---

## 5️⃣ Intruder - Attaques automatisées

- Configurer une attaque Intruder pour tester des payloads en masse (ex: bruteforce, fuzzing)  
> Permet d’automatiser les injections sur les paramètres vulnérables.

- Choisir le type d’attaque : Sniper, Battering ram, Pitchfork, Cluster bomb  
> En fonction du scénario et de la complexité du test.

- Utiliser des listes de payloads personnalisées ou intégrées (ex: fuzzdb, SecLists)  
> Pour maximiser la couverture des tests.

---

## 6️⃣ Decoder & Comparer

- Utiliser **Decoder** pour encoder/décoder rapidement des données (Base64, URL, Hex, etc.)  
> Pratique pour manipuler les données transmises.

- Utiliser **Comparer** pour voir les différences entre deux réponses ou requêtes  
> Utile pour détecter des changements subtils dans les réponses.

---

## 7️⃣ Extensions & automatisation

- Installer des extensions via **BApp Store** (ex: Logger++, Autorize, etc.)  
> Étendre les fonctionnalités de Burp Suite.

- Utiliser **Burp Suite API** pour automatiser des tâches (version Pro)  
> Intégration avec d’autres outils et scripts.

---

## 📌 Ressources utiles

- Documentation officielle : https://portswigger.net/burp/documentation  
- Tutoriels : https://portswigger.net/web-security  
- Payloads et listes : https://github.com/danielmiessler/SecLists  
- Communauté et forums : https://forum.portswigger.net/  
