# Network Defense

## 1. Le concept

### Où ça se situe dans NIST

La défense réseau relève de la fonction **Protect (PR)** du NIST CSF, notamment **PR.AC-5** (segmentation réseau) et **PR.PT-4** (sécurisation des communications). Le NIST recommande explicitement la segmentation comme contrôle de réduction d'impact : elle ne prévient pas la compromission initiale, mais elle limite ce que l'attaquant peut atteindre une fois entré.

### C'est quoi concrètement

La défense réseau combine trois leviers complémentaires :
- La **segmentation** : cloisonner le réseau pour qu'une compromission reste localisée
- Le **firewalling** : n'autoriser explicitement que les flux nécessaires
- L'**IDS/IPS** : détecter, voire bloquer, le trafic malveillant qui parvient malgré tout à circuler

## 2. Pourquoi ça marche (le mécanisme)

Sans segmentation, un réseau se comporte comme un grand espace ouvert : dès qu'un attaquant compromet une seule machine, il peut atteindre directement n'importe quelle autre ressource du réseau, y compris les serveurs les plus critiques. C'est ce qui permet le mouvement latéral rapide observé dans la majorité des intrusions réussies.

La segmentation change cette donne en imposant des points de passage obligés (les pare-feux inter-VLAN) entre les zones de criticité différente. Un poste utilisateur compromis reste alors cantonné à son propre segment, et toute tentative de le faire sortir de ce segment passe par un point de contrôle qui peut la bloquer ou au minimum la journaliser.

Un firewall en politique **deny-by-default** (tout est bloqué sauf ce qui est explicitement autorisé) inverse la charge de la preuve par rapport à une politique par défaut permissive : un flux non anticipé — souvent le signe d'une activité malveillante ou d'un mouvement latéral — est bloqué automatiquement, sans qu'il soit nécessaire de l'avoir identifié à l'avance comme dangereux.

Un IDS détecte sans bloquer (mode passif) ; un IPS agit en ligne sur le trafic et peut le bloquer en temps réel, au prix d'un risque de faux positif qui coupe du trafic légitime — d'où l'importance du calibrage.

## 3. Mise en œuvre — le chemin concret

### Segmenter le réseau

Découpage typique en zones de criticité croissante :
- VLAN Utilisateurs
- VLAN Serveurs applicatifs
- VLAN Infrastructure critique (contrôleurs de domaine, sauvegardes, administration)
- VLAN DMZ (services exposés à l'extérieur)

Principe : tout trafic inter-VLAN est bloqué par défaut, seuls les flux explicitement documentés dans une matrice de flux sont autorisés.

### Appliquer une politique de firewall restrictive

```bash
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT DROP

# Autoriser le trafic établi/retour
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A OUTPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Autoriser SSH uniquement depuis le VLAN admin
iptables -A INPUT -p tcp --dport 22 -s 10.10.99.0/24 -j ACCEPT
```

Les tentatives bloquées doivent être journalisées et envoyées vers le SIEM : un pic de connexions refusées vers plusieurs machines internes est souvent le signe d'un scan interne post-compromission.

### Déployer un IDS/IPS

Suricata en mode IPS (bloquant, en ligne sur le trafic) via NFQUEUE :
```bash
iptables -I FORWARD -j NFQUEUE --queue-num 0
suricata -c /etc/suricata/suricata.yaml -q 0
```

Calibrer les règles trop bruyantes pour éviter de bloquer du trafic légitime (identification par SID de la règle) :
```yaml
# /etc/suricata/disable.conf
2210050
```
```bash
suricata-update --disable-conf /etc/suricata/disable.conf
suricata-update
```

Limiter la répétition d'une même alerte pour éviter la saturation :
```
threshold gen_id 1, sig_id 2200003, type limit, track by_src, count 1, seconds 60
```

## Documentation officielle

- NIST Cybersecurity Framework : https://www.nist.gov/cyberframework
- Suricata : https://docs.suricata.io/en/latest/setup.html
- Netfilter/iptables : https://www.netfilter.org/documentation/
