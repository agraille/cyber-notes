---
tags:
  - blue-team
  - tool
---

# Process Hacker — Inspection & Forensics des processus

## C'est quoi

**Process Hacker** est un outil graphique Windows qui inspecte et manipule les processus en temps réel — bien plus puissant que le Task Manager natif. Utile pour :
- **Investigation** : voir exactement ce qu'un processus fait (DLLs chargées, handles ouverts, memory, connexions réseau)
- **Forensics** : confirmer une injection, un hijacking, une persistence
- **Sécurité** : terminer un processus malveillant, ou dumper sa mémoire

**Contre-partie** : outil que les malwares utilisent aussi pour s'injecter, dumper LSASS, etc.

---

## Installation & lancement

Télécharger : https://processhacker.sourceforge.io/

```
processhacker.exe
```

Lancer en administrateur pour accès complet (dump mémoire, manipulation processus).

---

## Inspection basique — l'interface

**Colonne principale** : liste de tous les processus avec :
- **Name** : nom du processus
- **PID** : identifiant unique
- **Priority** : priorité CPU (Normal, High, Low, Realtime)
- **CPU** : usage CPU actuel
- **Memory** : RAM utilisée
- **User** : compte propriétaire du processus (SYSTEM, administrateur, utilisateur standard, etc.)

Clic droit sur n'importe quel processus → menu contextuel avec 50+ options d'investigation.

---

## Onglets clés (clic droit → Properties)

### Onglet "General"

- **Name** : nom du process
- **PID** / **Parent PID** : arbre des processus
- **Base Priority** : priorité
- **User** : compte
- **Session ID** : numéro de session (Terminal Server)
- **Created** : heure de création du process
- **DEP** (Data Execution Prevention) : activé ou non

### Onglet "Memory"

```
Virtual Size : total mémoire que le process déclare
Working Set : RAM physique vraiment utilisée
Peak Working Set : max RAM utilisée depuis le démarrage
```

Très bruyant pour détecter une fuite mémoire (process qui grandit sans fin).

### Onglet "Modules" (DLL/Libraries chargées)

**C'EST LA CLUE POUR DÉTECTER** :
- **DLL Hijacking** : une DLL depuis un chemin anormal (Desktop au lieu de System32)
- **Injection .NET** : `clr.dll`, `mscoree.dll` chargées par un processus natif qui ne devrait pas
- **Injection de code** : DLL signée = false

Exemple inspection :
```
Explorer.exe charge :
  - kernel32.dll (Microsoft, System32) ✓ normal
  - shell32.dll (Microsoft, System32) ✓ normal
  - MALICIOUS.dll (unsigned, C:\Users\Public\Desktop) ✗ ROUGE
```

Clic droit sur une DLL → "Copy to file" pour la dumper et l'analyser statiquement.

### Onglet "Network"

Affiche toutes les **connexions réseau actives** du processus :

```
Process: explorer.exe
  Local Address : 192.168.1.50:5000
  Remote Address : 52.113.194.132:443
  State : ESTABLISHED
  Protocol : TCP
```

Détecte directement le beaconing C2, l'exfiltration réseau, la communication avec un serveur malveillant.

Clic droit → "Whois" pour info géographique rapide sur l'IP distante.

### Onglet "Threads"

Liste les **threads** du processus (chaque thread = une exécution parallèle).

Threads anormaux = injection probable. Un processus légitime a un nombre de threads prévisible.

### Onglet "Handles"

Fichiers, registre, événements, pipes **ouvert actuellement** par ce processus :

```
explorer.exe a ouvert :
  - C:\Windows\explorer.exe (fichier)
  - HKCU\Software\Microsoft\Windows (clé registre)
  - \\.\pipe\chrome_IPC (pipe nommé)
```

Utile pour savoir sur quoi un processus "regarde" en ce moment.

Clic droit → "Jump to" pour aller directement à la clé registre/fichier.

---

## Cas d'usage courants

### 1. Confirmer une injection de DLL

```
calc.exe devrait charger : kernel32, ntdll, msvcrt, ...
Tu vois dans Modules : MALICIOUS.dll depuis C:\Users\Public\Desktop

→ C'est un hijacking confirme
```

### 2. Détecter injection .NET (unmanaged PowerShell)

```
svchost.exe (processus natif) charge :
  - clr.dll (Microsoft .NET Runtime)
  - clrjit.dll
  
→ Anomalie = processus natif qui execute du code .NET
  = probable injection PowerShell non managée
```

### 3. Trouver un beaconing C2

```
Onglet Network : unknownprocess.exe 
  Connected to : 185.220.101.45:443 (TOR IP)
  Traffic : ESTABLISHED, 1MB/s up et down

→ C'est un beacon C2 actif
→ Dumper le processus, analyser statiquement, chercher les autres IOCs
```

### 4. Dumper la mémoire d'un processus malveillant

Clic droit sur processus → **"Miscellaneous"** → **"Dump Process"** (full dump, memory only, miniature dump)

Choisir le répertoire de sauvegarde. Ensuite analyser le dump avec Volatility ou strings.

### 5. Confirmer un mouvement latéral (code injection)

```
cmd.exe (Parent: svchost.exe - anormal)
   └─ Pas de ligne de commande (masqué)
   └─ Modules : clr.dll présent
   └─ Memory : Working Set très bas (suspicious)

→ Injection probable via svchost
→ Dumper la mémoire de cmd.exe pour forensics
```

---

## Commandes utiles

Clic droit sur processus → actions :

| Action | Usage |
|---|---|
| **Terminate** | Arrêter le processus (attention : peut crash système si critique) |
| **Suspend** | Geler le processus (pause sans terminer) — utile pour capturer l'état |
| **Dump Process** | Copier la mémoire pour analyse (Volatility, strings, etc.) |
| **Detach Debugger** | Si un debugger est attaché, le détacher |
| **Properties** | Accès à tous les onglets d'inspection |
| **Copy to File** | Pour les DLL : copier vers un dossier pour analyse statique |
| **Set Affinity** | Forcer le processus sur un CPU spécifique (debug) |
| **Token** | Voir/modifier les tokens (privilèges) du processus |
| **Virtual Memory** | Voir les zones mémoire du processus (RWX = injection) |

---

## Différencier processus légitime vs malveillant

| Signal | Légitime | Suspect |
|---|---|---|
| **Parent PID** | Logique (explorer.exe → cmd.exe) | Illogique (svchost.exe → cmd.exe) |
| **User** | SYSTEM/Admin si service, sinon utilisateur | SYSTEM lancant du code utilisateur |
| **Modules** | DLLs depuis System32, signées Microsoft | DLLs depuis Desktop/Temp, unsigned |
| **Memory** | Stable, croissance prévisible | Énorme, croissance rapide |
| **Network** | Peu/pas de connexions sortantes | Connexions réseau constantes à IPs bizarres |
| **Threads** | Nombre stable | Beaucoup de threads créés/détruits |
| **Handles** | Fichiers légitimes pour le process | Accès anormal (registre, pipes) |

---

## Intégration avec Sysmon/Windows Event Logs

Process Hacker = **inspection live**. Pour confirmation forensique, croiser avec :
- **Sysmon Event ID 7** : DLL loading (confirme l'injection)
- **Sysmon Event ID 8** : CreateRemoteThread (preuve d'injection)
- **Sysmon Event ID 10** : ProcessAccess (LSASS dumping)
- **Windows Event ID 4688** : process creation avec parent anormal

Workflow :
```
1. Process Hacker : "svchost.exe charge clr.dll - anomalie"
2. Sysmon Event 7 : confirme le chargement exact (timestamp, user)
3. Sysmon Event 8 : cherche CreateRemoteThread vers svchost
4. Dump mémoire + analyse statique
```

---

## Défense contre les abus

- **Restrict** : limiter l'accès à Process Hacker via GPO/AppArmor (attaquants s'en servent aussi)
- **Monitor** : détecter le lancement de Process Hacker (souvent outil d'intrusion)
- **Detect** : si Process Hacker accède à LSASS, c'est une alerte directe

---

## Documentation

- Process Hacker : https://processhacker.sourceforge.io/
- Documentation Process Hacker : https://processhacker.sourceforge.io/doc/index.php
