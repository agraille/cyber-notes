---
tags:
  - blue-team
  - defense
---

# Threat Hunting — Méthodologie & Bonnes Pratiques

## C'est quoi

Le **threat hunting** est une recherche **proactive et systématique** de compromissions qui n'ont **pas déclenché d'alerte automatique**. Contrairement à la détection classique (rule-based), ici c'est un humain qui formule l'hypothèse, définit la chasse, et trouve les preuves.

**Différence clé** :
- **Détection** : attendre qu'une règle se déclenche
- **Threat Hunting** : chercher activement ce que les règles ratent

**Enjeu** : tout attaquant capable cherche à contourner les défenses connues. Le threat hunting comble ce gap.

---

## Méthodologie Hypothesis-Driven (la fondation)

La bonne approche repose sur **5 étapes structurées** :

### Étape 1 : Formuler une hypothèse claire

Une hypothèse doit être :
- **Testable** : on peut chercher des preuves pour la confirmer/infirmer
- **Actuelle** : basée sur des menaces réelles et observables
- **Spécifique** : pas vague, pas générique

**Bonnes hypothèses** :
```
❌ Mauvais  : "On est attaqué"
✅ Bon     : "Un utilisateur execute PowerShell -enc depuis une machine autre que son poste habituel"

❌ Mauvais  : "Quelqu'un modifie des registre"
✅ Bon     : "Un process non-SYSTEM modifie HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\Defender"

❌ Mauvais  : "Il y a du mouvement latéral"
✅ Bon     : "Account Discovery (net view) suivi de Remote Service (psexec) par le même user dans une fenêtre de 10min"
```

### Étape 2 : Identifier les données nécessaires

Demande-toi : "Quels logs/données me permettront de tester cette hypothèse ?"

Exemple hypothèse : "PowerShell -enc exécuté hors des heures normales"

Données nécessaires :
- ✅ Event ID 4688 (process creation avec CommandLine détaillée)
- ✅ Horodatage précis
- ✅ Baseline des horaires de travail normaux par utilisateur
- ✗ Logs Sysmon (trop détaillés pour cette chasse)

### Étape 3 : Exécuter la chasse (requête)

Construire une requête sur le SIEM pour récupérer exactement ce dont tu as besoin.

Exemple Splunk SPL :

```spl
index=windows EventID=4688 AND CommandLine="*-enc*"
| eval hour=strftime(_time, "%H")
| where hour NOT IN (8,9,10,11,12,13,14,15,16,17)
| stats count by user, host, CommandLine
| where count > 0
```

Résultat = tous les PowerShell encodés en dehors des heures de travail.

### Étape 4 : Analyser les résultats

Classer les résultats en trois catégories :

**a) Confirmations légitimes**
```
user=admin_acme, 22:00, PowerShell -enc "C:\deploy\script.ps1"
→ Admin lance un déploiement la nuit = normal pour cette entreprise
```

**b) Faux positifs détectables**
```
user=john, 06:00, PowerShell -enc "Get-Help"
→ Get-Help est toujours encodé par défaut = faux positif
```

**c) Confirmations suspectes = ESCALADE EN INCIDENT**
```
user=jsmith, 02:15, PowerShell -enc "AAAA..." (inconnue)
computer=RANDOM-WORKSTATION-789
→ Escalader en incident, timeline complète, analyse mémoire
```

### Étape 5 : Documenter et capitaliser

**Chaque chasse = apprentissage** :

```
Hypothèse : PowerShell -enc hors heures
Résultat : 3 confirmations suspectes
    - Escaladé en incident X
    - Faux positif Get-Help (à exclure)
    - Faux positif admin cron job (à ajouter baseline)

Action : 
  - Créer une règle de détection automatique pour les futurs PowerShell -enc hors heures
  - Documenter les exceptions (admin deployments)
```

---

## Bonnes pratiques — La clé de la réussite

### 1. Structurer le programme de threat hunting

**Ne pas** : chasser au hasard, réagir aux alertes, ou suivre des rumeurs

**À faire** :
- Établir un **calendrier de chasses** (ex: lundi = lateral movement, mercredi = persistence)
- Couvrir les **tactiques MITRE ATT&CK** de façon systématique (évite les doublons)
- Documenter **chaque chasse** en cas de redécouverte
- **Prioriser** les chasses par risque (credential access avant discovery)

Exemple calendrier :
```
Semaine 1 : Initial Access (T1566 phishing, T1199 trusted relationship)
Semaine 2 : Execution (T1059 PowerShell, T1106 Native API)
Semaine 3 : Persistence (T1053 scheduled task, T1547 boot autostart)
Semaine 4 : Privilege Escalation (T1134 token impersonation, T1548 elevation bypass)
Mois 2 : Credential Access (T1110 bruteforce, T1003 LSASS dump)
Etc.
```

### 2. Impliquer l'équipe SOC (ne pas chasser seul)

**Bonnes pratiques** :
- **Briefing d'hypothèse** avant la chasse : présente à l'équipe, reçois du feedback
- **Pair hunting** : deux analystes explorent la même hypothèse (double validation)
- **Reviews post-chasse** : réunion avec le SOC pour discuter les résultats

Bénéfice : les collègues voient tes techniques, apprennent à chasser, augmentent la qualité des alertes automatiques.

### 3. Combiner chasses quantitatives et qualitatives

**Quantitative** (données massives) :
```
"Qui exécute PowerShell -enc ? Combien de fois ?"
Résultat : tableau avec user, count, IPs
```

**Qualitative** (investigation manuelle) :
```
"Pourquoi cet utilisateur exécute PowerShell -enc ?"
Deep-dive : timeline complète, parent process, réseau, fichiers modifiés
```

Les deux ensemble donnent une compréhension complète.

### 4. Utiliser MITRE ATT&CK pour valider l'hypothèse

Chaque hypothèse doit mapper à une ou plusieurs techniques MITRE.

Exemple :
```
Hypothèse : "Account Discovery (net view) suivi de Lateral Movement (psexec)"
    ↓
MITRE : T1087 (Account Discovery) + T1021 (Remote Services)
    ↓
Patterns typiques : APT28, APT29, Wizard Spider (vérifier les rapports)
    ↓
Prochaines étapes probables (selon MITRE) : T1005 Collection, T1041 Exfil
```

Ça te permet d'anticiper et de chasser les étapes suivantes.

### 5. Documenter les échecs aussi bien que les succès

**Les hypothèses infirmées sont des données** :

```
Hypothèse : "Lateral movement via WMI (Event ID 4688 wmiprvse.exe parent)"
Résultat : Aucun match dans 6 mois de logs
Conclusion : 
  - Pas de mouvement latéral WMI observé (élément de baseline sain)
  - Ou bien : les logs WMI ne sont pas collectés (gap de couverture)
  - Action : renforcer la collecte WMI pour les prochaines chasses
```

### 6. Itérer et affiner

Chaque chasse crée des données pour les **prochaines chasses**.

Exemple itération :
```
Chasse 1 : PowerShell -enc hors heures
  Résultat : 1 incident réel + 50 faux positifs admin

Chasse 2 (affinée) : PowerShell -enc hors heures EXCEPT admin users
  Résultat : 1 incident réel + 0 faux positifs
  
Chasse 3 (encore affinée) : PowerShell -enc hors heures, depuis machine jamais vue, user nouveau
  Résultat : 1 incident réel = pure signal
```

---

## Cycle complet d'une chasse (exemple réel)

### Jour 1 : Préparation

**Hypothèse** : "Un utilisateur crée une tâche planifiée (T1053) depuis un répertoire utilisateur, signe de persistence"

**Données nécessaires** :
- Event ID 4688 : schtasks.exe /create lancé
- Répertoire source : NOT System32 (detection d'anormalité)
- Validé par : Event ID 4698 (tâche planifiée créée)

### Jour 2 : Exécution

Requête Splunk :
```spl
index=windows (
  (EventID=4688 AND Image="*schtasks.exe" AND CommandLine="*/create*") 
  OR 
  EventID=4698
)
| where NOT CurrentDirectory IN ("C:\Windows\System32", "C:\Windows\SysWow64")
| stats count by user, host, CommandLine, EventID
| where count > 0
```

Résultats :
- 3 matches : tous provenant de System32 (faux positif détectable)
- 0 vrai suspect

### Jour 3 : Analyse

Raffinement :
```spl
index=windows EventID=4688 AND Image="*schtasks.exe" AND CommandLine="*/create*"
| where NOT CurrentDirectory IN ("C:\Windows\System32", "C:\Windows\SysWow64")
| where NOT user IN ("SYSTEM", "NT AUTHORITY")
| stats count by user, host
```

Résultat : aucun encore.

### Jour 4 : Itération & Documentation

**Conclusion** :
- Hypothèse infirmée pour cette période
- Collecte schtasks fonctionne correctement
- Baseline établi : 0 tâches planifiées créées par utilisateurs non-privilegié en dehors de System32

**Capitalisation** :
- Créer une règle de détection automatique : "Si schtasks /create EN DEHORS System32 → ALERTE"
- Document : "Task scheduler persistence hunting guide"

---

## Pièges à éviter

### ❌ Piège 1 : Hypothèse trop vague

```
Mauvais : "Chercher les choses bizarres"
Bon     : "Chercher les Process ID 4688 dont la CommandLine contient >3 redirects cmd"
```

### ❌ Piège 2 : Requête qui ne scale pas

```
Mauvais : SELECT * FROM all_logs WHERE anything_suspicious=true (crash SIEM)
Bon     : Filtrer d'abord par EventID précis, PUIS appliquer conditions
```

### ❌ Piège 3 : Chasse sans data

Avant de chercher, **valide que la data existe** :

```powershell
# Validation : commandLine collectée sur Event ID 4688 ?
Get-WinEvent -FilterHashtable @{ID=4688} -MaxEvents 1 | 
Select-Object -ExpandProperty Properties | 
Where-Object {$_.Name -like "*Command*"}
```

### ❌ Piège 4 : Ne pas documenter

La doc = la seule chose qui reste après toi. Sans doc, tu rechasses la même chose 6 mois plus tard.

### ❌ Piège 5 : Confondre "normal" et "légitime"

```
Exemple : Admin PowerShell -enc PENDANT les heures de travail = peut être normal (déploiement)
Mais : Admin PowerShell -enc vers une IP TOR = anormal MÊME si normal temporally
```

---

## Techniques de chasse avancées

### Technique 1 : Baseline + Anomaly Detection

Établir un baseline (état normal), puis chercher les écarts.

```
Baseline : 
  - Utilisateurs IT lancent PowerShell lun-ven 8h-18h
  - Moyenne 5 process création par personne/jour
  - 0 commandLine contenant "/c /s"

Chasse anomaly :
  - User non-IT lance PowerShell 23h (déviation)
  - User lance 50 process/jour (déviation)
  - CommandLine contient "/c /s" (déviation)
```

### Technique 2 : Temporal Correlation

Chercher deux événements liés dans le temps.

```
"Qui lance net view (T1087 discovery) PUIS psexec (T1021 lateral move) 
dans les 10 minutes suivantes ?"

Pour chaque net view :
  - Récupérer user, host, timestamp
  - Chercher psexec du même user dans [timestamp, timestamp+10min]
  - Si trouvé = lateral movement workflow
```

### Technique 3 : Threat Intelligence Pivoting

Utiliser MITRE ATT&CK pour trouver des patterns de groupe.

```
Hypothèse : "Je cherche les signatures APT28"
MITRE dit APT28 utilise : T1087 discovery, T1021 lateral, T1003 LSASS dump, T1041 exfil

Chasse APT28-like :
  - Chercher les 4 techniques dans 1 jour
  - Sur le même utilisateur/host
  - En ordre chronologique
```

### Technique 4 : Process Tree Anomaly

Chercher les arbres de processus illogiques.

```
Normal :
  explorer.exe
    └─ cmd.exe (utilisateur clique)
       └─ ipconfig.exe (commande lancée)

Suspect :
  System (PID 4)
    └─ svchost.exe (service légitime)
       └─ cmd.exe (svchost ne devrait pas lancer cmd)
          └─ powershell.exe -enc (cmd lancée de svchost, très suspect)
```

---

## Ressources & Outils

### Frameworks de chasse

- **MITRE ATT&CK** : https://attack.mitre.org/ (hypothèses basées sur techniques)
- **Cyber Threat Coalition Hunting Guide** : https://www.cisa.gov/
- **SANS Hunting Checklists** : SANS.org (acès limité)

### Outils SIEM

- **Splunk** : SPL queries pour threat hunting
- **Sentinel KQL** : requêtes KQL
- **ELK** : Kibana + Lucene queries
- **Wazuh** : dashboards threat hunting

### SOC Tools

- **Timeline Tools** : Plaso (disque), Volatility (mémoire)
- **Process Tools** : Process Hacker, SysInternals
- **Network Tools** : Zeek, tcpdump, Wireshark

---

## Points clés à retenir

1. **Hypothesis-driven = fondamental** : chaque chasse part d'une hypothèse testable
2. **MITRE ATT&CK = guide** : utilise les techniques pour valider/prédire
3. **Documenter tout** : hypothèse, résultats, exceptions, leçons
4. **Itération = clé** : affiner basé sur résultats
5. **Combiner quanti + quali** : données massives + investigation manuelle
6. **Capitaliser** : chaque chasse → règle de détection automatique potentielle
7. **Program, pas chaos** : calendrier de chasses structuré, pas réactif
