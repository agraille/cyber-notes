# Rédiger un rapport de pentest

Le rapport EST le livrable. Une exploitation parfaite avec un rapport flou ne vaut rien — ni pour un client, ni pour l'examinateur CJCA. Un rapport se juge sur la preuve, pas sur l'affirmation.

---

## C'est quoi l'objectif

Un rapport de pentest sert deux publics différents dans le même document :
- Un lecteur non-technique (management/client) qui doit comprendre le **risque business** sans jargon
- Un lecteur technique (équipe IT/dev) qui doit pouvoir **reproduire et corriger** la faille

Si une seule de ces deux cibles ne peut pas exploiter ton rapport, il est incomplet.

---

## Structure standard

### 1. Executive Summary (1 page max)

- Contexte : quoi, quand, périmètre testé
- Résultat en une phrase : posture de sécurité globale (bonne/moyenne/critique)
- Nombre de findings par sévérité (tableau ou graphique simple)
- Le risque principal à retenir — celui qui justifierait une action immédiate

Zéro jargon technique ici. Pas de nom de CVE, pas de payload. "Un attaquant externe peut accéder aux données clients sans authentification" — pas "SQLi sur le paramètre id via UNION-based blind".

### 2. Méthodologie

- Périmètre exact (IPs, domaines, comptes fournis, exclusions)
- Dates et fenêtre de test
- Type de test (boîte noire/grise/blanche)
- Standard suivi si applicable (PTES, OWASP Testing Guide)
- Limitations rencontrées (accès non fourni, service instable, etc. — protège aussi le testeur)

### 3. Findings (le cœur du rapport)

**Une fiche par vulnérabilité, avec ce squelette fixe :**

```
## [Titre clair et spécifique]

Sévérité : Critique / Haute / Moyenne / Basse / Info
CVSS : score + vecteur complet (ex: 9.8 - AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H)
CWE : référence si applicable

### Description
Ce qu'est la faille, en 2-3 phrases. Pas de cours sur le type de vuln en général.

### Impact
Ce qu'un attaquant peut concrètement faire avec. Spécifique à CETTE cible, pas générique.

### Preuve de concept
Étapes reproductibles, numérotées. Screenshot ou output brut à chaque étape clé.
Horodatage si pertinent (utile en partie blue/logs).

### Remédiation
Action concrète et priorisée. Pas "sécuriser le serveur" — plutôt
"Mettre à jour Dovecot vers la version X, désactiver AUTH PLAIN sans TLS".

### Références
CVE, lien CWE, doc officielle du correctif.
```

**Règles non négociables :**
- Un titre doit être compréhensible sans lire le reste ("Bypass d'authentification IMAP via absence de TLS forcé", pas "Faille IMAP")
- Une sévérité doit être justifiée par un score CVSS, jamais "à l'instinct"
- Zéro affirmation sans preuve — si tu ne peux pas le prouver, c'est une hypothèse, dis-le explicitement
- Une remédiation doit être actionnable par quelqu'un qui n'a pas fait le test

### 4. Priorisation / Plan d'action

Une matrice risque (probabilité × impact) qui reclasse les findings pour dire au client par quoi commencer — la sévérité technique seule ne suffit pas (une faille Critique sur un serveur de test isolé pèse moins qu'une Haute sur le serveur de prod exposé).

### 5. Annexes

- Sorties brutes des scans (nmap, burp, etc.)
- Liste des outils utilisés
- Logs bruts pertinents

---

## Partie spécifique : analyse de logs / détection (volet blue)

Si le rapport inclut une partie analyse de logs (SIEM, détection d'intrusion — typique d'un exercice hybride red/blue) :

- Chaque alerte statuée doit être **True Positive ou False Positive avec preuve**, jamais un jugement sans justification
- Format par alerte :
  ```
  Alerte : [nom/ID]
  Verdict : TP / FP
  Preuve : requête utilisée + extrait de log brut horodaté
  Justification : pourquoi ce verdict, en une phrase factuelle
  ```
- Ne jamais écrire "ça semble suspect" sans donner l'élément qui le rend suspect (IP source anormale, volume, timing, séquence d'événements)
- Citer les timestamps exacts — c'est ce qui permet de recorréler avec la partie offensive du rapport (ex: l'alerte à 14:32:07 correspond à l'exploitation documentée en section Findings)

---

## Erreurs qui font baisser la note/qualité d'un rapport

- Description vague ("le serveur est vulnérable") sans preuve reproductible
- Sévérité gonflée ou sous-estimée sans score CVSS pour justifier
- Copier-coller de description générique de vuln (type Wikipedia) au lieu de décrire l'impact réel sur la cible testée
- Remédiation générique non actionnable ("renforcer la sécurité")
- Incohérence entre l'Executive Summary et le détail technique (un exec summary qui minimise un risque documenté comme critique plus loin)
- Screenshots sans contexte (pas de commande visible, pas d'URL, pas d'horodatage)
- Fautes/structure qui cassent la lisibilité — un rapport se lit vite, il doit être scannable

---

## Ressources

- PTES (Penetration Testing Execution Standard) — méthodologie de référence
- OWASP Testing Guide — pour la partie web
- FIRST.org CVSS Calculator — scoring objectif
