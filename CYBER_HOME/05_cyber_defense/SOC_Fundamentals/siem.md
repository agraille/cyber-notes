---
tags:
  - blue-team
  - defense
  - domain/soc-fundamentals
---

# SIEM — Centralisation, corrélation, détection

## C'est quoi

Un **SIEM** (Security Information and Event Management) est une plateforme qui centralise, normalise et corrèle les logs/événements de toute l'infrastructure pour permettre la détection en temps réel et l'investigation forensique post-incident. C'est la couche de visibilité transverse qui relie SIEM → EDR → IDS → Logs applicatifs → Firewalls.

**Différence clé avec un simple log forwarder** :
- Log forwarder = envoie juste les logs vers un stockage central (passif)
- **SIEM = traite les logs, les corrèle, génère des alertes, permet la chasse (actif)**

**Points forts** :
- Visibilité 360° : un seul endroit pour chercher (pas de 50 interfaces différentes)
- Corrélation : relie des événements séparés pour révéler des patterns
- Alerting : génère des alertes exploitables, pas du bruit
- Forensics : permet de rejouer l'historique d'une attaque complet

**Limites** :
- Coûteux (ingestion + stockage + licences par GB/jour)
- Bruyant si les règles sont mal calibrées (alert fatigue)
- Dépendant de la qualité des logs collectés (garbage in, garbage out)

---

## Composants d'un SIEM

### Collecte (Forwarders/Agents)

Les logs **viennent** du terrain vers le SIEM :

```
Windows Event Logs → Forwarder (WEF / Splunk UF) → SIEM
Syslog (Linux/Unix) → Rsyslog/Syslog-ng → SIEM
Applications → HTTP/TLS vers API SIEM
Firewalls → Syslog/SNMP → SIEM
```

Exemple forwarder Windows (WEF) :

```powershell
# Configuration de forwarding Windows Event Logs vers SIEM (Splunk/Splunk UF)
New-Item -Path 'HKLM:\Software\Policies\Microsoft\Windows\EventLog\EventForwarding\SubscriptionManager' -Force
Set-ItemProperty -Path 'HKLM:\Software\Policies\Microsoft\Windows\EventLog\EventForwarding\SubscriptionManager' -Name '1' -Value 'Server=https://splunk-collector.domain.com:9997'
```

### Indexation

Les logs sont **parsés et stockés** dans un index interrogeable :

```
Raw log :
  "2024-01-15T10:23:45Z [ERROR] User john failed to authenticate (IP: 192.168.1.50)"

Indexé en champs :
  timestamp: 2024-01-15T10:23:45Z
  level: ERROR
  action: failed authentication
  user: john
  ip: 192.168.1.50
```

### Corrélation & Alerting

Les **règles de détection** cherchent des patterns :

```
Règle : "Bruteforce SSH"
  - Condition : 10+ failed SSH logins (Event ID 4625)
  - Source : même IP source
  - Fenêtre : dans 5 minutes
  - Action : créer une alerte

Règle : "Mouvement latéral"
  - Condition : Account Discovery (4688 net.exe) SUIVI DE Remote Service (4688 psexec)
  - Corrélation : même user, <10min
  - Action : alerte haute priorité
```

### Dashboard & Investigation

Interroger les logs de façon flexible :

```
"Montre-moi tous les Event ID 4624 depuis 192.168.1.50 
  dans les 2 dernières heures, groupés par utilisateur"

Résultat : tableau explorable, possibilité de drill-down
```

---

## Cas d'usage typiques (SIEM dans la vraie vie)

### 1. Détection en temps réel (alertes)

Règle : "PowerShell with encoded command"

```
EventID = 4688 AND Image = "powershell.exe" AND CommandLine = "*-enc*"
→ Trigger alerte immédiatement
```

### 2. Investigation post-incident (forensics)

Question : "Qu'a fait l'utilisateur jdupont pendant l'incident (28/01 10:00-15:00) ?"

```
Query :
  user = jdupont 
  AND TimeCreated >= 2024-01-28T10:00 
  AND TimeCreated <= 2024-01-28T15:00

Résultat : timeline complète de ses actions
```

### 3. Threat hunting (recherche proactive)

Hypothèse : "Qui exécute des process depuis le répertoire Temp ?"

```
EventID = 4688 AND CurrentDirectory = "*\Temp*" AND NOT Image = "trusted_process.exe"
→ Découvrir les processus suspectes
```

### 4. Compliance & Audit

Question : "Montre-moi les 10 derniers changements aux privilèges d'un compte sensible"

```
EventID = 4720 OR 4722 OR 4723 OR 4738
AND user IN ("Domain Admins", "Enterprise Admins", "Administrators")
```

---

## Choisir un SIEM

Trois catégories principales :

| SIEM | Coût | Facilité | Flexibilité |
|---|---|---|---|
| **Splunk** | Très cher | Facile | Très flexible |
| **Microsoft Sentinel** | Modéré (pay-as-you-go) | Facile | Flexible |
| **ELK Stack** (Elastic) | Gratuit (self-hosted) ou cloud | Moyen | Très flexible |
| **Wazuh** | Gratuit (open source) | Moyen | Flexible |
| **Sumo Logic** | Modéré (cloud-native) | Facile | Flexible |

**Critères de choix** :
- Budget : Splunk > Sentinel > Wazuh
- Volume logs : Splunk/Sentinel scalent bien, ELK/Wazuh dépend de l'infra
- Expertise interne : Splunk/Sentinel = plus de tutoriels, ELK/Wazuh = courbe plus raide

---

## Pipeline SIEM typique

```
Raw logs from infra
         ↓
[Collecteur]  ← Forwarder/Agent collecte
         ↓
[Transporteur] ← Envoie vers SIEM (TLS 1.2 min)
         ↓
[Récepteur] ← SIEM reçoit et parse
         ↓
[Normalisateur] ← Convertit en champs standards
         ↓
[Indexeur] ← Stocke et indexe pour recherche rapide
         ↓
[Corrélateur] ← Applique les règles de détection
         ↓
[Alerteur] ← Génère alertes si match
         ↓
[Tableau de bord] ← Visualisation pour l'analyste
```

---

## Structurer des logs pour le SIEM

### Exemple : Sysmon Event 1 (Process Create)

Log brut :
```
EventID=1|ProcessGuid={67e39d39-f72f-6269-6203-000000000300}|ProcessId=5560|CommandLine=powershell.exe -NoProfile -ExecutionPolicy Bypass -EncodedCommand AAAA|User=DOMAIN\user
```

Normalisé en champs SIEM :
```
timestamp: 2024-01-15T10:23:45Z
source_ip: 192.168.1.50
event_id: 1
event_type: process_create
process_name: powershell.exe
process_path: C:\Windows\System32\powershell.exe
command_line: powershell.exe -NoProfile -ExecutionPolicy Bypass -EncodedCommand AAAA
parent_process: explorer.exe
user: DOMAIN\user
computer: DESKTOP-ABC123
```

Une fois normalisé, tu peux créer une règle une seule fois et l'appliquer à **tous** les sources de logs (Splunk, Sentinel, ELK, etc.).

---

## Règles de détection dans un SIEM

### Format simple (Splunk SPL)

```spl
index=windows EventID=4624 OR EventID=4625
| stats count by src_ip, user
| where count > 10
| rename src_ip as Source, user as User
```

Cherche : 10+ logins (réussis ou échoués) depuis une même IP.

### Format complexe (corrélation)

```spl
index=windows EventID=4688 (Image="*\\net.exe" AND CommandLine="*view*")
| transaction user maxspan=10m
| search count > 1
| join type=inner user [search index=windows EventID=4688 Image="*\\psexec.exe"]
```

Cherche : net view (discovery) SUIVI DE psexec (lateral movement) par même user dans 10min.

---

## Points d'attention

- **Volume de logs ≠ valeur** : 1TB de logs inutiles coûte cher et ralentit les recherches — collecte seulement ce qui compte
- **Latence d'ingestion** : les alertes temps-réel sont rarement vraiment temps-réel (~5-30sec de latence)
- **Alert fatigue** : 100 alertes par jour que personne n'analyse = SIEM inutile — calibre les règles
- **Rétention des logs** : coûteux — définir une période (1an pour Splunk, 90j pour Sentinel cloud = standard)
- **Corrélation sur-calibrée** : trop de conditions = la règle ne trigger jamais (sous-détection)

---

## Ressources

- Splunk : https://www.splunk.com/
- Microsoft Sentinel : https://learn.microsoft.com/en-us/azure/sentinel/
- Elastic Stack (ELK) : https://www.elastic.co/
- Wazuh : https://wazuh.com/
- Sumo Logic : https://www.sumologic.com/
