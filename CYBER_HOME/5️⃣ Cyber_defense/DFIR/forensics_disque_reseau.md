# DFIR — Forensics Disque & Réseau

## 1. Le concept

### Où ça se situe dans NIST

Comme la forensics mémoire, cette activité s'appuie sur le **NIST SP 800-86**, qui structure la démarche d'investigation forensique en contexte de réponse à incident, et relève de la fonction **Respond (RS.AN — Analysis)** du NIST CSF. Là où la mémoire capture un instant présent, le disque et le réseau permettent de reconstruire le **déroulé temporel complet** d'une intrusion : quand l'attaquant est arrivé, ce qu'il a fait, dans quel ordre.

### C'est quoi concrètement

L'analyse disque reconstruit une chronologie précise des actions passées sur un système (fichiers créés, modifiés, exécutés — même après suppression). L'analyse réseau permet de rejouer et comprendre le trafic capturé pendant l'incident, pour savoir avec qui la machine compromise a communiqué et ce qui a été échangé.

## 2. Pourquoi ça marche (le mécanisme)

Un système de fichiers moderne ne se contente pas de stocker des fichiers : il maintient en permanence des métadonnées sur chaque action (date de création, de modification, d'accès), et Windows conserve en plus des traces d'exécution passées même si le programme a été supprimé depuis (le mécanisme de Prefetch, conçu à l'origine pour accélérer les lancements d'applications, devient ainsi une preuve d'exécution que l'attaquant ne pense pas toujours à effacer).

En alignant toutes ces métadonnées sur un seul axe temporel (une « timeline »), on peut reconstituer précisément la séquence d'une attaque, même longtemps après les faits, à condition que les traces n'aient pas été écrasées par une utilisation normale du disque.

Côté réseau, un pcap brut (capture de paquets) est difficile à exploiter directement à grande échelle. Des outils comme Zeek le retraitent en logs structurés par protocole (une ligne par connexion, par requête DNS, par transaction HTTP), ce qui permet de chercher des patterns (comme une exfiltration de données déguisée en requêtes DNS) bien plus rapidement qu'en lisant le trafic paquet par paquet.

## 3. Mise en œuvre — le chemin concret

### Construire une timeline disque avec Plaso

Extraire tous les artefacts temporels d'une image disque (MFT, Prefetch, Registre, logs d'événements) :

```bash
log2timeline.py --storage-file timeline.plaso /mnt/forensic/disk_image.dd
```

Générer une timeline lisible, filtrée sur la période de l'incident :

```bash
psort.py -o dynamic -w timeline.csv timeline.plaso \
  "date > '2026-06-20 08:00:00' AND date < '2026-06-20 12:00:00'"
```

### Artefacts Windows clés pour la timeline

- **MFT ($MFT)** : trace de création/modification/accès de tous les fichiers
- **Prefetch** (`C:\Windows\Prefetch\*.pf`) : preuve d'exécution d'un binaire, même supprimé depuis
- **Registre** : `NTUSER.DAT` pour l'activité utilisateur, `SYSTEM` pour les périphériques USB connectés (`SYSTEM\CurrentControlSet\Enum\USBSTOR`)
- **$UsnJrnl** : journal des changements NTFS, utile quand le fichier lui-même a été supprimé

```bash
pip install pyscca --break-system-packages
scca_dumper.py /mnt/forensic/mount/Windows/Prefetch/MALWARE.EXE-XXXXXXXX.pf
```

### Analyser une capture réseau avec Zeek

Zeek transforme un pcap en logs structurés exploitables directement :

```bash
zeek -r capture.pcap
# Génère conn.log, dns.log, http.log, files.log, ssl.log...
cat http.log | zeek-cut host uri user_agent
```

Repérer une exfiltration par tunneling DNS via la longueur anormale des requêtes (une exfiltration cache des données dans les sous-domaines interrogés, ce qui allonge fortement les requêtes) :

```bash
cat dns.log | zeek-cut query | awk '{ print length, $0 }' | sort -rn | head
```

### Extraction directe d'objets transférés

```bash
tshark -r capture.pcap --export-objects http,/mnt/forensic/extracted/
```

## Documentation officielle

- NIST SP 800-86 : https://csrc.nist.gov/pubs/sp/800/86/final
- Plaso / log2timeline : https://plaso.readthedocs.io/en/latest/
- Zeek : https://docs.zeek.org/en/master/
- Wireshark/tshark : https://www.wireshark.org/docs/
