# DFIR — Forensics Mémoire

## 1. Le concept

### Où ça se situe dans NIST

Le **NIST SP 800-86 (Guide to Integrating Forensic Techniques into Incident Response)** est la référence qui structure la démarche forensique dans un contexte de réponse à incident. Il insiste sur un principe central : **l'ordre de volatilité des données**. Certaines preuves disparaissent dès qu'on éteint ou redémarre une machine — la mémoire vive en fait partie. C'est pourquoi elle doit être capturée en tout premier, avant toute autre action d'investigation ou de remédiation.

Dans le NIST CSF, cette activité relève de la fonction **Respond**, catégorie **RS.AN (Analysis)** : comprendre précisément ce qui s'est passé pour pouvoir répondre efficacement.

### C'est quoi concrètement

L'analyse forensique mémoire consiste à examiner une copie de la RAM d'une machine compromise pour retrouver des artefacts qui n'existent jamais sur disque : processus injectés en mémoire, malware qui ne touche jamais le disque (« fileless »), clés de chiffrement encore présentes en mémoire, connexions réseau actives au moment exact de la capture.

## 2. Pourquoi ça marche (le mécanisme)

Un système d'exploitation maintient en permanence, en mémoire, l'état complet de ce qu'il exécute : la liste des processus, leurs connexions réseau, le code qu'ils ont chargé, y compris du code qui n'a jamais été écrit sur le disque. Un attaquant qui veut éviter la détection a intérêt à agir uniquement en mémoire, car les outils antivirus classiques scannent surtout le disque.

Ça crée une opportunité pour le défenseur : tant que la machine n'a pas été éteinte, cette activité invisible sur disque reste visible en mémoire. Un dump mémoire capturé au bon moment révèle donc des attaques qu'une simple analyse du disque ne verrait jamais — d'où l'urgence de le faire *avant* de redémarrer ou d'isoler brutalement une machine.

L'indicateur le plus révélateur d'une injection de code est la présence de zones mémoire marquées à la fois **inscriptibles et exécutables (RWX)** : un programme légitime n'a normalement pas besoin d'exécuter du code qu'il vient d'écrire lui-même en mémoire — c'est précisément ce que fait une injection.

## 3. Mise en œuvre — le chemin concret

### Acquisition de la mémoire (avant toute autre action)

```bash
# Linux - avec LiME (Linux Memory Extractor)
insmod lime.ko "path=/mnt/forensic/mem.lime format=lime"
```

```powershell
# Windows - avec WinPmem
winpmem.exe mem_dump.raw
```

### Identifier le système à partir du dump

```bash
python3 vol.py -f mem_dump.raw windows.info
```

### Lister les processus actifs

```bash
python3 vol.py -f mem_dump.raw windows.pslist
```

### Détecter des processus cachés

Un attaquant peut manipuler les structures internes de Windows pour masquer un processus à un simple listing. Comparer un scan de bas niveau (`psscan`) à l'arbre des processus (`pstree`) permet de repérer un écart révélateur :

```bash
python3 vol.py -f mem_dump.raw windows.psscan
python3 vol.py -f mem_dump.raw windows.pstree
```

### Lister les connexions réseau actives au moment du dump

```bash
python3 vol.py -f mem_dump.raw windows.netscan
```

### Détecter une injection de code

```bash
python3 vol.py -f mem_dump.raw windows.malfind
```

### Extraire un processus suspect pour analyse ultérieure

```bash
python3 vol.py -f mem_dump.raw windows.dlllist --pid <PID>
python3 vol.py -f mem_dump.raw -o ./output windows.pslist.PsList --pid <PID> --dump
```

### Artefacts à chercher en priorité

- Processus dont le parent est illogique (par exemple un lecteur de document qui lance un interpréteur de commandes)
- Processus sans chemin correspondant sur disque
- Zones mémoire RWX (indicateur fort d'injection de code)
- Connexions réseau vers des IP ou ports inhabituels

## Documentation officielle

- NIST SP 800-86 : https://csrc.nist.gov/pubs/sp/800/86/final
- Volatility 3 : https://volatility3.readthedocs.io/en/latest/
- LiME : https://github.com/504ensicsLabs/LiME
- WinPmem : https://github.com/Velocidex/WinPmem
