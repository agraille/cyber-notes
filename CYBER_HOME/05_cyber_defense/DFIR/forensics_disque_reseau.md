# DFIR — Forensics Disque & Réseau

## C'est quoi

**Forensics disque** : reconstruction d'une chronologie complète des actions passées sur un système (fichiers créés/modifiés/exécutés, même après suppression) via les métadonnées du système de fichiers.

**Forensics réseau** : rejouer et comprendre le trafic capturé pendant l'incident — qui a communiqué avec qui, quand, et quoi a été échangé.

**Enjeux** :
- Disque = timeline = preuve de quand/quoi s'est passé
- Réseau = preuve de C2, exfiltration, communication avec attaquant
- Combiné = reconstruire l'attaque complète du début à la fin

---

## Mise en œuvre — le chemin concret

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
