# Threat Intelligence — Pratique

## 1. Le concept

### Où ça se situe dans NIST

Le **NIST SP 800-150 (Guide to Cyber Threat Information Sharing)** structure la pratique du renseignement sur la menace, et dans le NIST CSF cette activité relève de la fonction **Identify (ID)**, catégorie **ID.RA (Risk Assessment)** : comprendre les menaces qui pèsent réellement sur l'organisation pour prioriser les défenses en conséquence, plutôt que de se défendre contre des menaces génériques.

### C'est quoi concrètement

La threat intelligence consiste à collecter, structurer et exploiter des informations sur les menaces (indicateurs de compromission, techniques d'attaque, campagnes actives) pour améliorer la détection et accélérer l'investigation. En pratique SOC, ça se traduit surtout par la centralisation d'indicateurs dans une plateforme dédiée (MISP est la référence open source) et leur exploitation active pendant une investigation.

## 2. Pourquoi ça marche (le mécanisme)

Un attaquant réutilise rarement une infrastructure entièrement nouvelle pour chaque intrusion : les mêmes IP, domaines, certificats SSL ou variantes de malware réapparaissent souvent sur plusieurs cibles, parce que construire une nouvelle infrastructure a un coût pour l'attaquant. C'est cette réutilisation qui rend le **pivoting** possible : partir d'un seul indicateur trouvé pendant une investigation (une IP suspecte, un hash de fichier) et remonter tout ce qui y est lié pour comprendre l'ampleur d'une campagne, voire l'identifier comme faite par un groupe connu.

Centraliser les indicateurs dans une plateforme partagée a aussi un effet réseau : un indicateur découvert par une organisation, partagé, devient immédiatement exploitable par d'autres avant même qu'elles ne soient touchées — c'est le principe qui sous-tend le NIST SP 800-150.

## 3. Mise en œuvre — le chemin concret

### Centraliser les indicateurs dans MISP

Ajouter un indicateur (hash malveillant) trouvé pendant une investigation :
```bash
curl -s --request POST \
  --url "https://misp.local/attributes/add/1234" \
  --header "Authorization: <API_KEY>" \
  --header "Content-Type: application/json" \
  --data '{
    "type": "sha256",
    "category": "Payload delivery",
    "value": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
    "to_ids": true
  }'
```

Vérifier si un indicateur est déjà connu avant de partir d'hypothèse zéro :
```bash
curl -s --request POST \
  --url "https://misp.local/attributes/restSearch" \
  --header "Authorization: <API_KEY>" \
  --header "Content-Type: application/json" \
  --data '{"value": "185.220.101.45"}'
```

### Pivoter sur un indicateur pour élargir la visibilité

Le principe : partir d'un indicateur unique et remonter les éléments liés, pour ne pas traiter chaque IOC de façon isolée.

1. **IP → domaines** : la passive DNS révèle quels domaines ont pointé vers cette IP dans le temps
```bash
curl -s "https://otx.alienvault.com/api/v1/indicators/IPv4/185.220.101.45/passive_dns" \
  -H "X-OTX-API-KEY: <API_KEY>"
```

2. **Domaine → certificats SSL** : un attaquant réutilise souvent le même certificat sur plusieurs domaines
```bash
curl -s "https://crt.sh/?q=%25.exemple-malveillant.com&output=json"
```

3. **Hash → famille de malware** : identifier la famille via une base de réputation permet de récupérer immédiatement les autres indicateurs déjà associés à cette famille (autres hashs, autres C2 connus)

4. **Techniques observées → groupe d'attaquants** : si les techniques utilisées correspondent à un mode opératoire déjà documenté, les autres techniques connues de ce même groupe deviennent des pistes de recherche immédiates dans l'environnement

### Sources ouvertes utiles pour enrichir une investigation

- **MalwareBazaar** (abuse.ch) : échantillons et hashs de malware
- **URLhaus** (abuse.ch) : URLs de distribution de malware actives
- **AlienVault OTX** : indicateurs partagés par la communauté

## Documentation officielle

- NIST SP 800-150 : https://csrc.nist.gov/pubs/sp/800/150/final
- MISP : https://www.circl.lu/doc/misp/
- MISP API : https://www.misp-project.org/openapi/
- AlienVault OTX : https://otx.alienvault.com/api
