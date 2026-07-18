---
tags:
  - blue-team
  - defense
  - domain/hardening
---

# Network Defense

## C'est quoi

La **défense réseau** combine trois leviers complémentaires :

1. **Segmentation** : cloisonner le réseau en zones pour qu'une compromission reste localisée
2. **Firewalling** : n'autoriser explicitement que les flux nécessaires (deny-by-default)
3. **IDS/IPS** : détecter/bloquer le trafic malveillant en temps réel

**Enjeu clé** : sans segmentation, une seule machine compromise = accès à **tout** le réseau. Avec segmentation, l'attaquant doit d'abord sortir de son VLAN avant d'avancer.

**Réalité** : la segmentation = le meilleur contrôle de lateral movement. Les IDS/IPS complètent mais ne remplacent pas.

---

## Mise en œuvre — le chemin concret

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
