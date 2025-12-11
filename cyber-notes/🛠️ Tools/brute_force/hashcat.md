🔥 Hashcat – Cheatsheet Cybersécurité

Hashcat est l’outil le plus performant pour cracker des hashs via GPU/CPU.
Cette cheatsheet couvre : extraction, modes d’attaque, formats, règles, masques, wordlists, etc.

🚀 1. Commandes de base

hashcat -h

Affiche l’aide complète

hashcat -I

Vérifie que les GPU sont bien détectés

hashcat -b

Benchmark de performances (utile pour choisir le meilleur mode)

🔍 2. Identifier un hash

hashid <hash>

Identifie automatiquement le type du hash

hash-identifier

Alternative interactive (Python)

🔢 3. Liste des modes Hashcat

hashcat --help | grep "Hash modes"

Liste complète des hash supportés

💡 Exemples courants :

0 → MD5

100 → SHA1

1400 → SHA256

1800 → SHA512

500 → md5crypt

1800 → sha512crypt

1000 → NTLM

13100 → Kerberos 5 TGS-REP (krb5tgs)

16800 → WPA/WPA2

16500 → JWT (HS256)

🛠 4. Attaque par dictionnaire

hashcat -m 0 -a 0 hash.txt rockyou.txt

Attaque simple via dictionnaire (MD5)

hashcat -m 1000 -a 0 ntlm.txt /usr/share/wordlists/*

Crack NTLM avec wordlists multiples

🔄 5. Attaque par masque (Bruteforce intelligent)

hashcat -m 0 -a 3 hash.txt ?l?l?l?l?l

5 lettres minuscules

hashcat -m 1000 -a 3 hash.txt ?u?l?l?d?d

Exemple : maj + minuscules + chiffres

Masques disponibles :

?l → lettre minuscule

?u → lettre majuscule

?d → chiffre

?s → symbole

?a → tous caractères imprimables

?h → hexadécimal (0-9 a-f)

🧠 6. Attaque combinée

hashcat -m 0 -a 1 hash.txt wordlist1.txt wordlist2.txt

Combine chaque mot de deux wordlists

🪓 7. Attaque basée sur des règles (Rules Attack)

hashcat -m 0 -a 0 hash.txt rockyou.txt -r rules/best64.rule

Applique des transformations aux mots du dictionnaire

hashcat -m 100 -a 0 hash.txt passwords.txt -r rules/dive.rule

Attaque agressive avec règles complexes

🧬 8. Attaque hybride (wordlist + masque)

hashcat -m 0 -a 6 hash.txt rockyou.txt ?d?d

Dictionnaire + suffixe de 2 chiffres

hashcat -m 0 -a 7 hash.txt rockyou.txt ?d?d

Préfixe 2 chiffres + dictionnaire

📡 9. Cracking de hashs spécifiques
🔐 Windows NTLM

hashcat -m 1000 -a 0 ntlm.txt rockyou.txt

Crack NTLM Windows

🧱 Kerberos (TGT / TGS)

hashcat -m 13100 -a 0 tgs.txt wordlist.txt

Kerberos TGS-REP (ASREPROAST)

📶 WPA/WPA2

hashcat -m 22000 capture.hc22000 rockyou.txt

WPA handshake converti au format 22000

🪙 JWT (HS256)

hashcat -m 16500 token.jwt rockyou.txt

Bruteforce de la clé secrète JWT

🧩 10. Pause / reprise / gestion des sessions

hashcat --session=mycrack -m 0 hash.txt rockyou.txt

Lance une session nommée

hashcat --restore

Reprend la dernière session arrêtée

hashcat --restore --session=mycrack

Reprise d’une session spécifique

hashcat -p :

Spécifie le délimiteur (utile pour fichiers hash:salt)

🧪 11. Format des fichiers hash
Format classique :

hash

Hash seul

Avec salt :

hash:salt

Hash + salt

Avec user:

user:hash

Nécessaire pour NTLM, Kerberos, etc.

🔧 12. Optimisations & Accélération

hashcat -O

Mode optimisé (plus rapide mais certaines attaques limitées)

hashcat --force

Ignore les warnings (à éviter sauf cas précis)

hashcat -w 3

Niveau de travail GPU (3 = agressif)

hashcat --status

Affiche statut en live

🧹 13. Nettoyage

hashcat --session=mycrack --restore

Reprise si arrêt accidentel

rm *.potfile

Supprime les hashs crackés stockés

history -c

Nettoyage des commandes utilisées

📚 14. Ressources utiles

Hashcat Wiki

Référence complète des modes et options

OneRuleToRuleThemAll

Règles très efficaces pour l’attaque rule-based

Rockyou.txt

Wordlist indispensable

SecLists

Wordlists pour attaques ciblées
