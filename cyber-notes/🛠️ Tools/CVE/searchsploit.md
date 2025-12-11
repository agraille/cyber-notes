# 💥 SearchSploit — Guide pratique étape par étape (Cybersécurité)

## 🔎 Cas pratiques (quand et pourquoi utiliser searchsploit)

> Veille vulnérabilité / Threat Intel

> Objectif : vérifier si un produit/version a des PoC publics pour prioriser un patch.

> Exemple : l’équipe sécurité reçoit une alerte CVE → on recherche un PoC pour évaluer le risque.

> Triage lors d’un pentest autorisé

> Objectif : trouver rapidement des exploits applicables pour prioriser des tests (lab).

> Exemple : cible string détectée WordPress 5.8.1 → chercher PoC pour plugins/themes.

> Préparation d’un labo de test

> Objectif : rassembler et organiser PoC pour reproduire des vulnérabilités dans des VMs isolées.

> Exemple : construire un pack d’exploits pour un exercice Red Team.

> Audit & conformité

> Objectif : confirmer l’existence ou non de PoC publics liés à une CVE pour justification d’urgence de patch.

> Exemple : tri par CVE-YYYY-NNNN.

> Recherche et analyse (reverse / forensics)

> Objectif : récupérer PoC pour analyser le mécanisme d’exploitation hors-ligne (sécurité académique).

# Mise à jour (paquet ou repo cloné)
```
searchsploit --update
```
# 🧾 Commandes de base (prêtes à copier / exécuter)

> Aide
```
searchsploit --help
```
> Rechercher un terme simple
```
searchsploit apache
```
> Rechercher une expression exacte
```
searchsploit "remote code execution"
```
> Rechercher par CVE
```
searchsploit CVE-2021-44228
```

# 🔬 Commandes selon les cas d'usage (pratiques)
## Cas 1 — Veille CVE (trouver rapidement si un PoC public existe)
> Recherche directe par CVE
```
searchsploit CVE-2023-XXXXX
```
> Si vous voulez aussi afficher en sortie plus verbale (filtrer)
```
searchsploit CVE-2023-XXXXX | less
```
## Cas 2 — Triage rapide pour un produit/version trouvé dans l’inventaire
> Rechercher product + version
```
searchsploit "wordpress 5.8.1"
```
> Filtrer les résultats pertinents (exemples : rce, remote, overflow, privilege)
```
searchsploit "wordpress 5.8.1" | grep -Ei "remote|rce|overflow|priv|exploit"
```
## Cas 3 — Préparer un paquet d’exploits pour le labo
> Cloner le dépôt (si pas déjà fait)
```
git clone https://github.com/offensive-security/exploitdb.git
```
> Rechercher dans le dépôt cloné et copier les PoC correspondant à "nginx"
```
grep -Ri "nginx" exploitdb | awk -F: '{print $1}' | sort -u | xargs -I{} cp {} ~/lab/exploits/
```
> Exemple plus robuste (sélectionner seulement les fichiers dans exploits/)
```
grep -Ril "nginx" exploitdb/exploits | xargs -I{} cp {} ~/lab/exploits/
```
## Cas 4 — Triage rapide pendant un pentest : récupérer un PoC localement (sans exécuter)
> Voir les résultats et copier le PATH affiché
```
searchsploit "php 7.4" | less
```

# 🧰 Commandes avancées / astuces pratiques
> Filtrer insensible à la casse
```
searchsploit mysql | grep -i "overflow"
```
> Extraire uniquement le champ PATH (adapter selon votre version de searchsploit)
```
searchsploit openssl | awk -F'/' '{print $0}'  # -> adapter pour récupérer la colonne PATH
```
> Ouvrir la page web Exploit-DB correspondante (si supporté)
```
searchsploit -w "CVE-2020-1234"
```
> Lister uniquement les EDB-IDs (selon output)
```
searchsploit nginx | grep -oE "EDB-ID: [0-9]+" | sort -u
```
> Rechercher plusieurs mots-clés (ET)
```
searchsploit "php rce" | grep -i "php" | grep -i "rce"
```

⚠️ Le format exact de la sortie peut varier selon la version de searchsploit. Adaptez les awk/sed/grep aux colonnes affichées sur ta machine.

# 🗂️ Workflow complet — Exemple pas-à-pas (Triage pentest)

> Identifier la cible et la version (ex. http-server X.Y.Z)

> Rechercher PoC pertinents :
```
searchsploit "http-server X.Y.Z" | less
```
> Filtrer pour les types critiques (RCE, auth bypass) :
```
searchsploit "http-server X.Y.Z" | grep -Ei "remote|rce|bypass|auth"
```
> Copier les PoC sélectionnés dans ~/lab/exploits/ pour analyse :
```
cp "/usr/share/exploitdb/exploits/XXXX/nom_poc.py" ~/lab/exploits/
```

Inspecter manuellement chaque PoC (lire le code, comprendre les pré-conditions) — NE PAS EXÉCUTER EN PROD.

Documenter : EDB-ID, titre, CVE (si présent), chemin local, résumé, risques, recommandations.


# ✅ Résumé des outils utiles

searchsploit (Exploit-DB)

git, grep, awk, sed, less

nc, jq (selon besoins d’automatisation)

Environnement VM/sandbox pour tests

# 📚 Références & ressources rapides (à consulter pour approfondir)

Dépôt officiel Exploit-DB : https://github.com/offensive-security/exploitdb

Exploit-DB (site) : https://www.exploit-db.com/

searchsploit --help (aide locale)
