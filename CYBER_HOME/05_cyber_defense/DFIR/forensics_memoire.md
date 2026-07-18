---
tags:
  - blue-team
  - defense
---

# DFIR — Forensics Mémoire

## C'est quoi

L'analyse forensique mémoire consiste à examiner une copie de la RAM d'une machine compromise pour retrouver des **artefacts invisibles sur disque** :
- Processus injectés (en mémoire seulement)
- Malware fileless (jamais écrit sur disque)
- Clés de chiffrement en mémoire
- Connexions réseau actives au moment exact

**Clé** : la mémoire est **volatile** — elle disparaît au redémarrage. C'est pourquoi on doit la capturer en **premier**, avant toute autre action.

**IOC clé** : zones mémoire RWX (ReadWriteExecute) = signe d'injection de code.

---

## Mise en œuvre — le chemin concret

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
