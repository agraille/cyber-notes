---
tags:
  - blue-team
  - defense
  - domain/log-windows
---

# Sysmon — Détection d'activité malveillante

## C'est quoi
 
**Sysmon** (System Monitor) est un service Windows + driver de périphérique qui reste résident entre les redémarrages pour surveiller et journaliser l'activité système en profondeur. Contrairement aux logs natifs Windows qui sont limités, Sysmon fournit une visibilité bien plus riche sur la création de processus, les connexions réseau, les chargements de DLL, l'accès aux autres processus, etc.
 
- Fournit des **Event IDs propres** (1-28) catégorisés par type d'activité (1=Process Create, 3=Network Connection, 7=Image Loaded, 10=Process Access...)
- Logs dans le journal standard Windows : `Applications and Services Logs > Microsoft > Windows > Sysmon > Operational`
- Configurable via fichier XML pour inclure/exclure certains types d'événements (exemple : exclure les logs de certains processus légitimes trop bruyants)
- Clé pour détecter : injection de code, chargement de .NET en processus natif, accès à LSASS (credential dumping), DLL hijacking, beaconing réseau
**Intérêt pentest / défense** :
- Détecte ce que Windows natif ne voit pas (pas de chargement de DLL par défaut, pas de hachage de fichier, pas d'arbre process complet)
- Un fichier `.evtx` Sysmon exporté = preuve d'activité, idéal pour forensics/post-incident
- Sysmon + Get-WinEvent = combinaison puissante pour chasser les attaques discrètes (injection, DLL hijacking, .NET injection)
---

## Installation

```cmd
sysmon.exe -i -accepteula -h md5,sha256,imphash -l -n
```

Appliquer une configuration personnalisée (recommandé : base SwiftOnSecurity) :

```cmd
sysmon.exe -c sysmonconfig-export.xml
```

Configs de référence :
- https://github.com/SwiftOnSecurity/sysmon-config (config complète, prête à l'emploi)
- https://github.com/olafhartong/sysmon-modular (approche modulaire par technique)

Logs consultables dans : `Observateur d'événements > Applications and Services Logs > Microsoft > Windows > Sysmon > Operational`

## Table des Event IDs Sysmon utiles
 
| Event ID | Type | Usage détection |
|---|---|---|
| 1 | Process Create | Création de process, avec ligne de commande et hash |
| 2 | File Creation Time Changed | Timestomping — un attaquant modifie la date de création pour masquer un artefact |
| 3 | Network Connection | Connexions réseau sortantes (C2, exfiltration) |
| 4 | Sysmon Service State Changed | Sysmon démarré/arrêté — un arrêt est un signal fort de tentative d'evasion |
| 5 | Process Terminated | Fin de process — corrélation avec l'Event ID 1 pour reconstituer une timeline complète |
| 6 | Driver Loaded | Chargement de driver — détection de rootkits ou d'outils type Mimikatz-driver / BYOVD |
| 7 | Image Loaded | Chargement de DLL — clé pour DLL hijacking et détection .NET |
| 8 | CreateRemoteThread | Injection de code dans un autre process |
| 9 | RawAccessRead | Lecture brute d'un disque (contournement des ACL fichiers) — typique de dump de SAM/NTDS |
| 10 | Process Access | Accès à un process (LSASS pour credential dumping) |
| 11 | File Create | Création de fichier |
| 12 | Registry Event (Create/Delete) | Création/suppression de clé de registre — persistance, désactivation de sécurité |
| 13 | Registry Value Set | Persistance via registre |
| 14 | Registry Event (Rename) | Renommage de clé/valeur registre — évasion de règles basées sur un nom fixe |
| 15 | FileCreateStreamHash | Création d'Alternate Data Stream — souvent lié au Mark of the Web (fichier téléchargé) |
| 16 | Sysmon Config Change | Modification de la configuration Sysmon — signal fort de tentative de tampering par l'attaquant |
| 17 | Pipe Created | Création de named pipe — utilisé par certains frameworks C2 (Cobalt Strike, etc.) |
| 18 | Pipe Connected | Connexion à un named pipe — corrélation avec l'Event ID 17 |
| 19 | WMI Event Filter | Création d'un filtre WMI — persistance WMI (WMI subscription) |
| 20 | WMI Event Consumer | Création d'un consumer WMI — persistance WMI, exécution de payload au déclenchement |
| 21 | WMI Event Consumer To Filter | Liaison filtre/consumer WMI — complète la chaîne de persistance WMI |
| 22 | DNS Query | Requêtes DNS, utile pour C2/tunneling |
| 23 | File Delete (archived) | Suppression de fichier avec copie archivée par Sysmon — anti-anti-forensics |
| 24 | Clipboard Change | Changement du presse-papier — exfiltration de données, credential harvesting |
| 25 | Process Tampering | Process hollowing / herpaderping — technique d'évasion avancée |
| 26 | File Delete (logged) | Suppression de fichier journalisée sans archivage (léger, sans copie) |
| 27 | File Block Executable | Blocage d'un exécutable par Sysmon selon règles de config |
| 28 | File Block Shredding | Blocage d'une tentative d'écrasement/déchiquetage de fichier |
| 29 | File Executable Detected | Détection d'écriture d'un fichier exécutable sur disque |

Activer l'Event ID 7 (désactivé par défaut car bruyant) — dans le XML de config, passer le `RuleGroup ImageLoad` de `include` à `exclude` (sans règle d'exclusion = tout est capturé) :

```xml
<RuleGroup name="" groupRelation="or">
  <ImageLoad onmatch="exclude">
  </ImageLoad>
</RuleGroup>
```

---

## Cas 1 — Détection de DLL Hijacking

**Principe de l'attaque** : remplacer une DLL légitime attendue par un exécutable par une DLL malveillante du même nom, placée dans un répertoire où l'exécutable la chargera en priorité (ordre de recherche Windows).

**Reproduction (exemple calc.exe / WININET.dll)** :

```cmd
:: Renommer la DLL réflective en WININET.dll
ren reflective_dll.x64.dll WININET.dll

:: Copier calc.exe + WININET.dll dans un dossier accessible en écriture (ex: Desktop)
copy C:\Windows\System32\calc.exe C:\Users\Public\Desktop\
copy WININET.dll C:\Users\Public\Desktop\

:: Exécuter
C:\Users\Public\Desktop\calc.exe
```

**Détection — filtrer Sysmon Event ID 7 sur calc.exe** :

Observateur d'événements > Filter Current Log > Event ID 7, puis rechercher "calc.exe".

**IOC à corréler (les 3 ensemble confirment le hijacking)** :
1. `calc.exe` chargé depuis un répertoire hors `System32`/`Syswow64` (jamais normal)
2. `WININET.dll` chargée par `calc.exe` depuis un chemin hors `System32` (le nom ne peut pas être changé par l'attaquant sans casser le hijack, donc c'est un IOC fiable)
3. Champ `Signed` = `false` sur la DLL chargée (l'originale est signée Microsoft)

**Récupérer le hash de la DLL malveillante** :
```powershell
Get-FileHash C:\Users\Public\Desktop\WININET.dll -Algorithm SHA256
```

---

## Cas 2 — Détection d'injection PowerShell/C# non managée

**Principe** : injecter du bytecode C#/PowerShell dans un processus natif (non-.NET) via un runtime CLR chargé dynamiquement, pour exécuter du code managé dans un process qui n'en a normalement pas besoin (execute-assembly, unmanaged PowerShell).

**Reproduction (exemple avec PSInject dans spoolsv.exe)** :

```powershell
powershell -ep bypass
Import-Module .\Invoke-PSInject.ps1
Invoke-PSInject -ProcId <PID_spoolsv.exe> -PoshCode "V3JpdGUtSG9zdCAiSGVsbG8sIEd1cnU5OSEi"
```

**Détection** :

Le processus injecté passe d'un état natif à un état "managed (.NET)". Vérifiable via Process Hacker (onglet Modules) ou directement via Sysmon Event ID 7 en cherchant le chargement de :
- `clr.dll`
- `clrjit.dll`

dans un processus qui ne les charge normalement jamais (`spoolsv.exe`, `svchost.exe`, etc.).

**IOC** : présence de `clr.dll`/`clrjit.dll` (Microsoft .NET Runtime) chargées par un processus natif Windows qui n'a aucune raison légitime d'exécuter du code managé.

**Récupérer le hash de la DLL chargée** :
```powershell
Get-FileHash "C:\Windows\Microsoft.NET\Framework64\v4.0.30319\clrjit.dll" -Algorithm SHA256
```

---

## Cas 3 — Détection de Credential Dumping (LSASS)

**Principe** : extraire les identifiants en mémoire depuis LSASS (Local Security Authority Subsystem Service), typiquement via Mimikatz.

**Reproduction** :
```cmd
mimikatz.exe
privilege::debug
sekurlsa::logonpasswords
```

**Détection — Sysmon Event ID 10 (ProcessAccess)** :

Filtrer sur `TargetImage` = `lsass.exe`.

**IOC à corréler** :
1. `SourceImage` inhabituel (exécutable depuis `Downloads`, `Temp`, ou tout chemin non standard) accédant à `lsass.exe`
2. `SourceUser` différent de `TargetUser` (ex: un utilisateur standard accédant à un process tournant en `SYSTEM`)
3. Le processus source a demandé/obtenu `SeDebugPrivilege` juste avant (nécessaire pour lire la mémoire d'un autre process)

**Note** : des process légitimes (AV, EDR, outils d'authentification) accèdent aussi à LSASS normalement — c'est la combinaison chemin d'exécutable + privilèges + contexte utilisateur qui distingue un accès légitime d'un dump.

**Récupérer un hash NTLM depuis la sortie Mimikatz** :
Le hash apparaît directement dans la sortie de `sekurlsa::logonpasswords` sous `msv > NTLM`.

## Documentation

- Sysmon (Microsoft Sysinternals) : https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon
- Sysmon Event ID reference complète : https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon#events
- SwiftOnSecurity config : https://github.com/SwiftOnSecurity/sysmon-config
- sysmon-modular : https://github.com/olafhartong/sysmon-modular
