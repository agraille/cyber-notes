---
tags:
  - blue-team
  - defense
---

# Incident Handling Process (PICERL)

## C'est quoi

La gestion d'incidents est le processus structuré qui permet à un SOC de détecter, contenir, éradiquer et récupérer suite à une compromission, puis d'en tirer des leçons. L'objectif : transformer une attaque chaotique en procédure maîtrisée, en minimisant l'impact et le temps de résolution (MTTD/MTTR).

Le cycle de vie standard (SANS) est découpé en 6 phases, souvent désigné par l'acronyme **PICERL** :

1. Preparation
2. Identification
3. Containment
4. Eradication
5. Recovery
6. Lessons Learned

---

## Méthodologie

### 1. Preparation

Phase proactive, avant tout incident. Objectif : être prêt à réagir vite et bien.

- Playbooks de réponse par type d'incident (ransomware, phishing, exfiltration...)
- Outils en place : SIEM (Splunk, ELK, Wazuh), EDR (CrowdStrike, Defender for Endpoint), outils forensics (Velociraptor, KAPE)
- Golden image / baseline de référence pour comparer un système suspect à un état sain
- Contacts d'urgence (CERT, legal, management) et matrice RACI définie
- Test régulier via tabletop exercises ou purple team

Exemple : forwarder des logs Linux vers un SIEM via rsyslog

```bash
# /etc/rsyslog.d/siem-forward.conf
*.* @@siem.local:514
```

### 2. Identification

Détecter un événement suspect et confirmer s'il s'agit d'un incident réel (vs faux positif).

- Sources : alertes SIEM/EDR, IDS (Suricata, Snort), logs Windows (Sysmon, Event ID 4624/4625/4688), NetFlow
- Corrélation avec des IOCs connus (hash, IP, domaine C2)

Exemple de requête Sysmon (recherche de process suspect via Event ID 1) :

```
EventID=1 AND (Image="*\\powershell.exe" AND CommandLine="*-enc*")
```

Recherche rapide de connexions sortantes suspectes avec `ss` :

```bash
ss -tnp | grep ESTAB
```

Vérification d'un hash contre une base de réputation (VirusTotal API) :

```bash
curl -s --request GET \
  --url "https://www.virustotal.com/api/v3/files/<hash>" \
  --header "x-apikey: <API_KEY>"
```

### 3. Containment

Empêcher la propagation. On distingue souvent confinement à court terme (immédiat) et long terme (stabilisation).

- Isolation réseau d'un host via EDR, ou manuellement :

```bash
# Isoler une IP compromise au niveau firewall
iptables -A INPUT -s <ip_compromise> -j DROP
iptables -A OUTPUT -d <ip_compromise> -j DROP
```

- Désactivation d'un compte AD compromis :

```powershell
Disable-ADAccount -Identity jdupont
```

- Isolation d'un poste au niveau switch (port shutdown) ou VLAN de quarantaine
- Snapshot / image disque avant toute action destructive (préservation de la preuve)

```bash
dd if=/dev/sda of=/mnt/forensic/host01.img bs=4M status=progress
```

### 4. Eradication

Éliminer la cause racine, pas juste les symptômes.

- Suppression du malware identifié, correction de la vulnérabilité exploitée (patch, reconfiguration)
- Rotation complète des credentials potentiellement exposés
- Recherche de persistance (tâches planifiées, clés Run, services)

```powershell
# Tâches planifiées suspectes
Get-ScheduledTask | Where-Object {$_.TaskPath -notlike "\Microsoft\*"}

# Clés de persistance classiques
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Run"
```

```bash
# Vérification cron / persistence Linux
crontab -l
cat /etc/cron.d/*
```

### 5. Recovery

Retour à la normale, de façon sécurisée et surveillée.

- Restauration depuis backup sain (vérifié hors ligne, non compromis)
- Reconstruction complète si la confiance dans le système ne peut être garantie
- Remise en production progressive avec monitoring renforcé (surveillance des IOCs pendant plusieurs jours/semaines)

```bash
rsync -avz --checksum /mnt/backup_clean/ /srv/app/
```

### 6. Lessons Learned

Post-mortem après clôture de l'incident (idéalement sous 2 semaines).

- Timeline complète de l'incident documentée dans TheHive ou équivalent
- Root cause analysis
- Mise à jour des playbooks, des règles de détection (SIEM) et des contrôles préventifs
- Rapport formel si obligation réglementaire (RGPD, NIS2)

---

## Outils & concepts clés

- **TheHive** — plateforme SIRP (Security Incident Response Platform) pour centraliser cases, alertes et observables. Doc : https://docs.thehive-project.org/
- **MISP** — plateforme de partage et gestion d'IOCs, s'intègre avec TheHive. Doc : https://www.misp-project.org/documentation/
- **Cyber Kill Chain** (Lockheed Martin) — modèle en 7 étapes décrivant le déroulement d'une attaque (Reconnaissance → Actions on Objectives), utile pour situer où on peut détecter/casser la chaîne. Doc : https://www.lockheedmartin.com/en-us/capabilities/cyber/cyber-kill-chain.html
- **MITRE ATT&CK®** — base de connaissances des tactiques/techniques (TTPs) réelles des attaquants, permet de mapper un incident à des techniques connues (ex: T1059 - Command and Scripting Interpreter). Doc : https://attack.mitre.org/
- **IOC (Indicator of Compromise)** — preuve observable d'une intrusion : hash de fichier, IP/domaine C2, clé de registre, mutex...
- **IOA (Indicator of Attack)** — se concentre sur le comportement/intention de l'attaquant plutôt que sur des artefacts statiques, plus résistant à l'évasion.

## Sources

- SANS Incident Handler's Handbook — https://www.sans.org/white-papers/33901/
- NIST SP 800-61 Rev.2 (Computer Security Incident Handling Guide) — https://csrc.nist.gov/pubs/sp/800/61/r2/final
