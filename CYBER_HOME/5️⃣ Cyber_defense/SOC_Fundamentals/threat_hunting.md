# Threat Hunting

## 1. Le concept

### Où ça se situe dans NIST

Le threat hunting relève de la fonction **Detect** du NIST CSF, catégorie **DE.AE (Anomalies and Events)**, mais dans sa forme la plus mature : au lieu d'attendre qu'une règle automatique déclenche une alerte, l'analyste part chercher activement des signes de compromission qui n'ont *pas* déclenché d'alerte. Le NIST considère la détection continue et l'amélioration itérative comme un objectif central — le threat hunting en est l'expression la plus proactive.

**Point clé** : toute règle de détection a des angles morts (une technique qu'elle ne couvre pas, une variante qu'elle ne reconnaît pas). Le threat hunting part du principe qu'un attaquant compétent peut passer sous les règles existantes, et cherche à le débusquer manuellement.

### C'est quoi concrètement

Le threat hunting est une recherche proactive de compromission, pilotée par un humain, qui part d'une hypothèse plutôt que d'une alerte automatique. C'est la différence fondamentale avec la détection classique : ici, l'humain décide où chercher, au lieu de réagir à ce qu'une règle a déjà repéré.

## 2. Pourquoi ça marche (le mécanisme)

Les attaquants avancés savent que les défenseurs utilisent des règles de détection connues, et adaptent leurs techniques pour les éviter (utilisation d'outils légitimes déjà présents sur le système plutôt que de malware détectable, fragmentation des actions dans le temps pour éviter les seuils d'alerte). Une approche purement basée sur des règles automatiques ne peut, par définition, détecter que ce qu'elle a été programmée à reconnaître.

Le threat hunting comble ce vide en posant une hypothèse fondée sur des techniques d'attaque connues ("si un attaquant voulait faire du mouvement latéral sans se faire repérer, comment s'y prendrait-il sur ce système ?") puis en allant chercher des preuves de cette hypothèse dans les données brutes, même en l'absence d'alerte. Si l'hypothèse se confirme, elle devient une piste d'investigation immédiate ; si elle échoue, elle enrichit la connaissance du terrain pour la prochaine chasse.

## 3. Mise en œuvre — le chemin concret

### La démarche hypothesis-driven

1. **Formuler une hypothèse** précise, par exemple : "un attaquant utilise l'injection de processus pour évader les outils de détection"
2. **Identifier les données nécessaires** pour vérifier cette hypothèse (quels logs, quelle télémétrie permettent de l'infirmer ou de la confirmer)
3. **Chasser** : exécuter des requêtes ciblées sur les données identifiées
4. **Conclure** : si une preuve est trouvée, la chasse devient une investigation d'incident ; sinon, on documente et on affine l'hypothèse suivante
5. **Capitaliser** : si le pattern cherché s'avère être une menace réelle et récurrente, transformer la requête manuelle en règle de détection automatisée, pour ne plus avoir à la rechasser à la main

### Exemples de requêtes de chasse

**Injection de processus** — repérer un process qui écrit dans la mémoire d'un autre process que lui-même (Sysmon Event ID 8) :
```
EventID=8 AND SourceImage!=TargetImage
```

**Mouvement latéral via PsExec** — recherche d'un service temporaire au nom caractéristique de l'outil :
```powershell
Get-WinEvent -FilterHashtable @{LogName='System'; ID=7045} |
  Where-Object { $_.Message -match 'PSEXESVC' }
```

**Mouvement latéral via WMI** — un parent process `wmiprvse.exe` menant à une exécution suspecte :
```
EventID=4688 AND ParentImage="*\wmiprvse.exe" AND NOT Image="*\WmiPrvSE.exe"
```

**Living-off-the-land** — usage de binaires légitimes (PowerShell notamment) avec des arguments d'obfuscation :
```
EventID=4688 AND (CommandLine="*-enc*" OR CommandLine="*-EncodedCommand*")
```

**Beaconing réseau** — connexions périodiques et régulières vers une même destination externe, signe typique d'un C2 :
```spl
index=netflow dest_ip!=<plage_interne>
| bucket _time span=1h
| stats count by _time, dest_ip, src_ip
| where count > 20
```

## Documentation officielle

- NIST SP 800-137 (ISCM, surveillance continue) : https://csrc.nist.gov/pubs/sp/800/137/final
- LOLBAS project (référence des binaires légitimes détournables sous Windows) : https://lolbas-project.github.io/
