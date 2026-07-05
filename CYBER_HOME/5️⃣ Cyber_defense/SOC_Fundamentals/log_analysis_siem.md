# Log Analysis & SIEM

## 1. Le concept

### Où ça se situe dans NIST

Dans le **NIST Cybersecurity Framework (CSF)**, le SIEM appartient à la fonction **Detect (DE)** :
- **DE.CM (Security Continuous Monitoring)** : surveiller en continu les systèmes pour repérer des événements de sécurité
- **DE.AE (Anomalies and Events)** : détecter une activité anormale et en comprendre l'impact

Dans le **NIST SP 800-61 Rev.2**, le SIEM est l'outil qui rend possible la phase d'identification d'un incident. Sans lui, la détection repose sur la chance plutôt que sur un processus fiable.

**Point clé** : on ne peut pas détecter ce qu'on ne collecte pas. Avant même de choisir un outil, la vraie question est : *quelles sources de données couvrent mes scénarios de menace les plus probables ?* Un SIEM avec de mauvaises sources de logs ne sert à rien, aussi cher soit-il.

### C'est quoi concrètement

Un SIEM (Security Information and Event Management) fait trois choses :
1. **Collecte** les logs de toute l'infrastructure (serveurs, postes, firewalls, applications) dans un endroit centralisé
2. **Normalise** ces logs (des formats hétérogènes deviennent des champs comparables : IP source, utilisateur, timestamp...)
3. **Corrèle** les événements entre eux pour transformer des logs isolés en alertes exploitables

### Vocabulaire à maîtriser

- **Événement** : n'importe quelle action journalisée (un logon, une connexion réseau). Il y en a des millions par jour, la plupart bénins.
- **Alerte** : un événement (ou une combinaison d'événements) qui correspond à une règle de détection et mérite un regard humain.
- **Incident** : une alerte confirmée comme étant une véritable compromission ou violation de politique de sécurité.

Le rôle du SIEM est de réduire le volume : passer de millions d'événements à quelques dizaines d'alertes triables par un analyste humain.

## 2. Pourquoi ça marche (le mécanisme)

Une attaque complète est presque toujours multi-étapes : reconnaissance, delivery, exploitation, mouvement latéral, exfiltration. Chaque étape touche un système différent et laisse une trace différente :

- La reconnaissance laisse une trace dans les logs firewall/proxy (scan de ports)
- L'exploitation laisse une trace dans les logs applicatifs ou EDR
- Le mouvement latéral laisse une trace dans les logs d'authentification
- L'exfiltration laisse une trace dans les logs réseau (volume de données sortant anormal)

Pris séparément, chacun de ces logs peut sembler anodin. C'est la **corrélation dans le temps et entre sources** qui révèle le pattern d'attaque. Un humain ne peut pas croiser manuellement des millions de lignes de logs provenant de dizaines de systèmes différents — un moteur de corrélation le peut.

Le problème inverse existe aussi : si les règles de corrélation sont trop larges, on génère trop d'alertes et l'analyste finit par les ignorer (« alert fatigue »). Le calibrage des règles est aussi important que leur existence.

## 3. Mise en œuvre — le chemin concret

### Étape 1 — Identifier les sources de logs critiques

**Windows (via Sysmon + Event Log natif)**
- Event ID 4624 / 4625 : logon réussi / échoué
- Event ID 4688 : création de process (avec ligne de commande si l'audit est activé)
- Event ID 4768 / 4769 : demandes de tickets Kerberos
- Sysmon Event ID 1 (création process), 3 (connexion réseau), 11 (création fichier) — Sysmon comble les manques du Event Log natif (pas de hash de fichier, pas d'arbre process complet par défaut)

**Linux**
- `/var/log/auth.log` (Debian/Ubuntu) ou `/var/log/secure` (RHEL) : authentifications
- `auditd` (`/var/log/audit/audit.log`) : traçabilité fine des actions système

**Réseau**
- Logs firewall/proxy : souvent la meilleure source pour repérer C2 et exfiltration
- NetFlow : visibilité sur les volumes et destinations de trafic

### Étape 2 — Activer une collecte suffisamment détaillée

Par défaut, Windows ne loggue pas la ligne de commande des process créés. Il faut l'activer explicitement :

```powershell
auditpol /set /subcategory:"Process Creation" /success:enable
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\Audit" /v ProcessCreationIncludeCmdLine_Enabled /t REG_DWORD /d 1
```

Sur Linux, forwarder les logs vers le SIEM via syslog :

```bash
# /etc/rsyslog.d/siem-forward.conf
*.* @@siem.local:514
```

### Étape 3 — Interroger et corréler

Une fois les logs centralisés, la compétence à développer est d'écrire des requêtes de recherche : filtrer, agréger, seuiller. La syntaxe change selon l'outil mais la logique de fond est toujours la même.

**Exemple : détecter un bruteforce (logons échoués répétés depuis une même IP)**

Splunk (SPL) :
```spl
index=windows EventCode=4625
| stats count by src_ip, Account_Name
| where count > 10
```

Microsoft Sentinel (KQL) :
```kql
SecurityEvent
| where EventID == 4625
| summarize FailedCount = count() by IpAddress, Account
| where FailedCount > 10
```

ELK (Kibana Query Language) — le filtrage se fait ici, l'agrégation ensuite dans une visualisation :
```
event.code:4625 and source.ip:*
```

### Étape 4 — Industrialiser

Une requête ponctuelle, c'est de l'investigation manuelle ad hoc. L'étape suivante consiste à transformer les requêtes qui fonctionnent en règles automatiques, versionnées et testées, pour détecter en continu sans réécrire la logique à chaque fois.

## Documentation officielle

- NIST SP 800-61 Rev.2 : https://csrc.nist.gov/pubs/sp/800/61/r2/final
- NIST Cybersecurity Framework : https://www.nist.gov/cyberframework
- Splunk SPL : https://docs.splunk.com/Documentation/Splunk/latest/SearchReference/WhatsInThisManual
- KQL (Sentinel) : https://learn.microsoft.com/en-us/azure/sentinel/kusto-overview
- Sysmon : https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon
