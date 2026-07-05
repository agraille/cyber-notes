# Detection Engineering

## 1. Le concept

### Où ça se situe dans NIST

Le NIST Cybersecurity Framework classe cette activité dans **DE.DP (Detection Processes)**, sous la fonction Detect : le NIST demande explicitement que les processus de détection soient testés, maintenus et améliorés en continu — pas juste "activés une fois". Le **NIST SP 800-137 (Information Security Continuous Monitoring)** formalise cette idée de surveillance continue et itérative plutôt que statique.

**Point clé** : une règle de détection n'est jamais définitive. Les techniques d'attaque évoluent, les faux positifs s'accumulent, l'infrastructure change — une règle non maintenue devient obsolète ou bruyante en quelques mois.

### C'est quoi concrètement

Le detection engineering consiste à écrire, tester et faire évoluer des règles de détection de façon systématique et versionnée (« detection-as-code »), plutôt que de créer des alertes une par une, à la main, sans suivi.

Deux familles de règles couvrent des sources différentes :
- **Règles basées sur les logs/hôtes** (format Sigma) : détectent un comportement à partir de ce qui se passe *sur une machine* (création de process, accès mémoire...)
- **Règles basées sur le réseau** (format Suricata) : détectent un comportement à partir du *trafic qui circule* (paquets, signatures, patterns HTTP...)

## 2. Pourquoi ça marche (le mécanisme)

Une règle créée une seule fois, à la main, dans l'interface d'un outil, se perd avec le temps : personne ne sait pourquoi elle existe, elle n'est jamais mise à jour, et si elle génère trop de faux positifs on la désactive... et on oublie de la réactiver. En traitant les règles comme du code versionné (Git, revue, tests), on garde une trace de leur raison d'être, on peut les tester avant de les déployer en production, et on peut les porter facilement d'un outil à un autre.

Le format Sigma illustre bien ce principe : la règle est écrite une seule fois dans un format neutre, puis convertie automatiquement vers la syntaxe propre à chaque SIEM. Ça évite de réécrire la même logique de détection plusieurs fois pour plusieurs outils, avec le risque d'incohérence que ça implique.

## 3. Mise en œuvre — le chemin concret

### Écrire une règle Sigma (détection basée logs)

Exemple : détection d'un dump de LSASS, technique classique de vol de credentials en mémoire.

```yaml
title: Suspicious LSASS Process Access
status: stable
logsource:
    category: process_access
    product: windows
detection:
    selection:
        TargetImage|endswith: '\lsass.exe'
        GrantedAccess:
            - '0x1010'
            - '0x1410'
            - '0x1438'
    condition: selection
level: high
```

Conversion vers une requête Splunk avec `sigma-cli` :

```bash
pip install sigma-cli --break-system-packages
sigma convert -t splunk lsass_dump.yml
```

### Écrire une règle Suricata (détection réseau)

Exemple : détection d'un User-Agent HTTP suspect, potentiellement associé à un beacon C2.

```
alert http any any -> any any (msg:"Suspicious User-Agent - possible C2 beacon"; \
  http.user_agent; content:"curl/"; \
  classtype:trojan-activity; sid:1000001; rev:1;)
```

Test de syntaxe avant déploiement (une règle mal écrite peut faire planter le moteur en prod) :

```bash
suricata -T -c /etc/suricata/suricata.yaml -S custom.rules
```

### Faire vivre les règles dans le temps

1. La règle est écrite et versionnée dans un dépôt Git
2. Elle est testée contre des logs/pcap de référence pour vérifier qu'elle détecte bien ce qu'elle doit détecter (et rien d'autre)
3. Elle passe par une revue avant déploiement
4. Une fois en production, son taux de faux positifs est surveillé et la règle est ajustée si besoin
5. Sa couverture est cartographiée par rapport aux techniques d'attaque connues, pour savoir explicitement quelles techniques sont détectées et lesquelles ne le sont pas encore

## Documentation officielle

- NIST SP 800-137 (ISCM) : https://csrc.nist.gov/pubs/sp/800/137/final
- Sigma : https://sigmahq.io/docs/
- Sigma rule repository : https://github.com/SigmaHQ/sigma
- Suricata rules : https://docs.suricata.io/en/latest/rules/
