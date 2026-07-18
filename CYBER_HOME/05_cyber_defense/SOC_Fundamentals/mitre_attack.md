---
tags:
  - blue-team
  - defense
---

# MITRE ATT&CK — Classification & Prédiction d'actions

## C'est quoi

**MITRE ATT&CK** est une base de connaissances librement accessible qui documente les tactiques et techniques que les attaquants **réels** utilisent. C'est une grille de référence mondiale pour classifier ce qu'on observe et prédire la suite d'une attaque.

Structure : **Tactiques** (objectifs haut niveau : Execution, Persistence) contiennent des **Techniques** (T1234 = identifiant unique).

**Intérêt** :
- Classifier ce qu'on observe (Event ID 4688 + PowerShell -enc = T1059.001)
- Prédire la suite (si T1059 Execution, le prochain est probablement T1087 Discovery)
- Mesurer la couverture de détection (combien de techniques on détecte vs combien existent)

---

## Tactiques principales (12 phases d'attaque)

| Tactique | Exemple technique |
|---|---|
| Initial Access | T1566 (Phishing), T1190 (Exploit Public App) |
| Execution | T1059 (PowerShell), T1106 (Native API), T1053 (Scheduled Task) |
| Persistence | T1053 (Scheduled Task), T1547 (Boot Autostart) |
| Privilege Escalation | T1548 (Elevation Bypass), T1134 (Token Impersonation) |
| Defense Evasion | T1562 (Disable Logging), T1140 (Deobfuscate Code) |
| Credential Access | T1110 (Brute Force), T1003 (LSASS Dump), T1187 (Forced Auth) |
| Discovery | T1087 (Account Discovery), T1082 (System Information), T1057 (Process Discovery) |
| Lateral Movement | T1021 (Remote Services), T1570 (Tool Transfer) |
| Collection | T1005 (Local Data), T1123 (Audio Capture) |
| Exfiltration | T1041 (Exfil over C2), T1020 (Auto-exfiltration) |
| Command & Control | T1071 (Application Layer Protocol), T1008 (Fallback Channels) |
| Impact | T1531 (Account Removal), T1561 (Disk Wipe) |

---

## Mapper des événements Windows à MITRE

### Cas 1 : PowerShell encodé (Event ID 4688)

Observé dans les logs :
```
powershell.exe -NoProfile -ExecutionPolicy Bypass -EncodedCommand AAAA...
```

Mapping MITRE :
```
Technique : T1059.001 (PowerShell)
Tactique : Execution
```

Prochaines étapes probables :
- T1087 (Account Discovery) — énumérer les comptes
- T1057 (Process Discovery) — scanner processus
- T1082 (System Information Discovery)
- T1105 (Ingress Tool Transfer) — télécharger un outil

### Cas 2 : Accès à LSASS (Sysmon Event ID 10)

Observé dans Sysmon :
```
SourceImage: ProcessHacker.exe
TargetImage: C:\Windows\System32\lsass.exe
GrantedAccess: 0x1400
```

Mapping MITRE :
```
Technique : T1003.001 (LSASS Memory Dump)
Tactique : Credential Access
```

Prochaines étapes probables :
- T1110 (Brute Force) — craquer les hashs extraits
- T1550.002 (Pass-the-Hash) — réutiliser les hashs
- T1021 (Remote Services) — connexion distante avec les creds

### Cas 3 : Tâche planifiée (Event ID 4698)

Observé :
```
schtasks.exe /create /tn "Windows Update Check" /tr calc.exe
```

Mapping MITRE :
```
Technique : T1053.005 (Scheduled Task)
Tactique : Persistence + Execution
```

Prochaines étapes probables :
- T1005 (Data from Local System) — exfiltration fichiers
- T1082 (System Information Discovery)
- T1021 (Lateral Movement)

---

## Chaîne d'attaque typique (Kill Chain MITRE)

La séquence prévisible d'une attaque :

```
Initial Access (T1566: Phishing)
         ↓
Execution (T1059: PowerShell)
         ↓
Defense Evasion (T1562: Disable Logging, T1548: Elevation Bypass)
         ↓
Persistence (T1053: Scheduled Task)
         ↓
Privilege Escalation (T1134: Token Impersonation)
         ↓
Credential Access (T1003: LSASS Dump)
         ↓
Discovery (T1087: Account Discovery, T1057: Process Discovery)
         ↓
Lateral Movement (T1021: Remote Services, T1570: Tool Transfer)
         ↓
Collection (T1005: Local Data, T1025: Data from Removable Media)
         ↓
Exfiltration (T1041: Exfil over C2, T1020: Auto-exfiltration)
         ↓
Impact (T1531: Account Access Removal, T1561: Disk Wipe)
```

**Clé** : Si tu as observé une tactique, les prochaines étapes sont prédictibles. Utilise ça pour la chasse proactive.

---

## Threat Hunting basée sur MITRE

### Hypothèse de chasse structurée

Hypothèse : "Si on observe T1087 (Account Discovery) suivi de T1021 (Remote Services) dans une fenêtre de 10 min par le même processus, c'est un lateral movement."

Technique :
```powershell
# 1. Chercher les Event IDs associés à T1087 (Account Discovery)
# Typiquement : net.exe, dsquery, quser, whoami, Get-ADUser
Get-WinEvent -FilterHashtable @{ID=4688} |
Where-Object {$_.Message -like "*net view*" -or $_.Message -like "*dsquery*"}

# 2. Pour chaque match, chercher T1021 dans les 10 min suivantes
# Typiquement : svcctl, wmi, ssh, psexec, etc.
# Event IDs : 4688 (process), 4648 (RunAs), 4720 (net use)

# 3. Corréler par processus/user/source
# Résultat = détection lateral movement
```

---

## Utiliser MITRE pour mesurer la couverture détection

### Matrice de couverture

Quelles techniques tu détectes **réellement** avec ton SIEM/EDR ?

```
Execution
  - T1059 PowerShell ✓ (Event ID 4688 + filtre "-enc")
  - T1106 Native API ✗ (pas de couverture)
  - T1053 Scheduled Task ✓ (Event ID 4698)

Credential Access
  - T1003 LSASS Dump ✓ (Sysmon Event ID 10)
  - T1110 Brute Force ✓ (Event ID 4625)
  - T1187 Forced Authentication ✗ (pas de couverture)

Defense Evasion
  - T1562 Disable Logging ✗ (pas de couverture)
  - T1548 Elevation Bypass ✗ (pas de couverture)
  - T1036 Masquerading ✗ (compliqué à détecter)

Couverture totale : 6/12 techniques détectables = 50%
```

**Usage** : Identifie les angles morts (techniques non détectées) et priorise les règles de détection.

---

## Identifier un groupe APT par ses techniques

Chaque groupe d'attaquants a une signature MITRE — ses techniques préférées.

Exemples (fictifs) :
- **APT28** : T1059 PowerShell + T1003 LSASS Dump + T1021 Remote Services
- **APT29** : T1566 Phishing + T1204 User Execution + T1078 Valid Accounts
- **Wizard Spider** : T1486 Ransomware (Impact) + T1020 Auto-exfil

Si tu observes une séquence de techniques :

```
Techniques observées = {T1003, T1021, T1087, T1590}
         ↓
Recherche MITRE (attack.mitre.org/groups/)
Quels groupes utilisent CETTE combinaison de techniques ?
         ↓
Possibilité : APT28, APT29, ou groupe similaire
         ↓
Consulte les rapports de ces groupes pour prédire les prochaines étapes
```

---

## Naviguer MITRE ATT&CK

### Site officiel : attack.mitre.org

- **Par tactique** : `/tactics/` → clique Execution → vois toutes les techniques T10xx
- **Par groupe** : `/groups/` → clique APT28 → vois leurs 50+ techniques documentées
- **Par plateforme** : `/matrices/` → Enterprise (Windows/Linux/Cloud), Mobile, ICS

### MITRE ATT&CK Navigator (visualisation)

Outil : https://mitre-attack.github.io/attack-navigator/

Crée ta propre matrice :
1. Code couleur par "détecté" (vert) vs "non-détecté" (rouge)
2. Visualise ta couverture du SIEM en coup d'œil
3. Partage avec l'équipe pour identifier collectivement les gaps

---

## Points clés à retenir

- **Nomenclature partagée mondiale** : T1003 = même chose pour tous
- **Les chaînes d'attaque suivent des patterns** : si T1087, attends T1021
- **MITRE mesure la couverture** : "Je détecte 50 techniques sur 600" est quantifiable
- **Chaque groupe a une signature MITRE** : techniques peuvent aider à identifier l'attaquant
- **Aligné au monde réel** : MITRE = observations de vraies attaques, pas théorie

---

## Ressources

- MITRE ATT&CK Framework : https://attack.mitre.org/
- MITRE ATT&CK Navigator : https://mitre-attack.github.io/attack-navigator/
- Groupes APT documentés : https://attack.mitre.org/groups/
- Techniques par domaine : https://attack.mitre.org/matrices/enterprise/
