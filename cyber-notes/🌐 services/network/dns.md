# 🌐 DNS - Cheatsheet CyberSécurité

Ce fichier contient les **commandes de base**, **outils d'attaque**, et **techniques de cracking/exploitation DNS**, spécialement utiles en pentest.

---

## 🔍 1. Enumération de base

dig domaine.com  
> Résolution classique (A record)

nslookup domaine.com  
> Interroge le DNS par défaut

host domaine.com  
> Résout le domaine

dig NS domaine.com  
> Récupère les serveurs de noms (Nameservers)

dig MX domaine.com  
> Récupère les serveurs mails (Mail Exchanger)

host -t txt domaine.com  
> Cherche des infos dans les enregistrements TXT (souvent SPF, DKIM)

---

## 🕵️ 2. Découverte de sous-domaines

dnsenum domaine.com  
> Enumération DNS complète + brute-force + transfert de zone

dnsmap domaine.com  
> Enumération de sous-domaines rapide

gobuster dns -d domaine.com -w wordlist.txt  
> Brute-force DNS avec wordlist

amass enum -d domaine.com  
> Enumération massive de sous-domaines (recon OSINT + bruteforce)

subfinder -d domaine.com  
> Enumération passive via API publiques

---

## 📡 3. Résolution inversée

dig -x IP  
> PTR record (Reverse DNS)

host IP  
> Alternative simple pour reverse lookup

---

## 🔐 4. Transfert de zone (AXFR)

dig AXFR domaine.com @ns1.domaine.com  
> Tente un transfert de zone complet (grave faille si ouvert)

host -l domaine.com ns1.domaine.com  
> Teste un transfert via `host`

> ⚠️ Si ça fonctionne, tous les sous-domaines et IP internes sont exposés

---

## 💣 5. Attaque DNS Cache Snooping

dig @cible.com google.com  
> Si la réponse est instantanée → domaine déjà résolu → présence d'un client interne

> Utile pour savoir quels domaines sont visités par les utilisateurs d’un résolveur DNS

---

## 🧪 6. Brute-force / Fuzzing DNS

dnsrecon -d domaine.com -t brt  
> Fuzz des sous-domaines

massdns -r resolvers.txt -t A -o S -w result.txt wordlist.txt  
> Résolution massive ultra rapide

---

## 🧠 7. Scripts Python utiles

# Résolution simple de tous les enregistrements

import dns.resolver  
domain = "example.com"  
for qtype in ["A", "NS", "MX", "TXT"]:  
 answers = dns.resolver.resolve(domain, qtype)  
 for r in answers:  
  print(f"{qtype} -> {r.to_text()}")

---

## 🔓 8. Exploitation & cracking

dig +short txt domaine.com  
> Cherche des clés, secrets, metadata exposées

findomain -t domaine.com -u result.txt  
> Trouve des sous-domaines et les enregistrements liés

fierce -dns domaine.com  
> Enumération complète + scan IP + sous-domaines

cewl http://domaine.com -w wordlist.txt  
> Génère une wordlist basée sur le contenu du site → utile pour brute-force DNS ou accès

---

## 📚 9. Outils à connaître

- `dig`, `host`, `nslookup` → Résolution DNS
- `dnsenum`, `fierce`, `dnsrecon` → Enumération active
- `amass`, `subfinder`, `assetfinder` → Recon passive
- `massdns`, `findomain` → Résolution massive
- `cewl`, `wfuzz` → Fuzz / wordlist

---

## 🛡 10. Défense / Durcissement

- Bloquer le transfert de zone (AXFR)
- Masquer les sous-domaines sensibles (pas d'entrée DNS publique)
- Monitorer le trafic DNS sortant
- Utiliser DNSSEC pour éviter le spoofing
- Limiter les requêtes à des clients autorisés

---

## 🧭 Ressources

- https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/DNS%20Reconaissance  
- https://owasp.org/www-community/attacks/DNS_Rebinding  
- https://github.com/blechschmidt/massdns  
