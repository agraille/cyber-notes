📧 SMTP – Cheatsheet Cybersécurité

Cheatsheet complète pour analyser, énumérer, tester, abuser, et exploiter les serveurs SMTP.
Utilisable en pentest interne/externe, CTF, formation SOC, etc.

🔍 1. Scan & Détection du service SMTP

nmap -p 25,465,587 <IP>

Détection des ports SMTP classiques : 25 (non chiffré), 465 (SMTPS), 587 (submission)

nmap --script smtp-commands -p 25 <IP>

Récupération des commandes SMTP supportées

nmap --script smtp-enum-users -p 25 <IP>

Tentative d’énumération des utilisateurs SMTP

nmap --script smtp-vuln* -p 25 <IP>

Recherche de vulnérabilités connues (par ex. Open Relay)

📡 2. Test de connexion SMTP en manuel

telnet <IP> 25

Connexion manuelle SMTP non chiffrée

nc <IP> 25

Alternative à telnet

openssl s_client -connect <IP>:465

Connexion TLS/SSL (SMTPS)

openssl s_client -starttls smtp -connect <IP>:587

Connexion via STARTTLS

🧪 3. Énumération d’utilisateurs (VRFY / EXPN / RCPT)
🔎 Test VRFY

VRFY admin

Vérifie si l’utilisateur existe

VRFY root

Test des comptes système classiques

🔎 Test EXPN

EXPN staff

Liste les membres d’une mailing-list

🔎 RCPT TO brute-force (fallback)

MAIL FROM:test@test.com

RCPT TO:user@domain.com

Test d’existence via réponse 250/550

🧰 4. Commandes SMTP utiles en clair

HELO test

Présentation au serveur (version non-chiffrée)

EHLO test

Présentation + affichage des extensions SMTP

MAIL FROM:attacker@evil.com

Définition de l’expéditeur

RCPT TO:victim@domain.com

Définition du destinataire

DATA

Début du contenu du mail

.

Fin du contenu et envoi

QUIT

Déconnexion propre

🪓 5. Test Open Relay

MAIL FROM:test@external.com

Adresse externe

RCPT TO:victim@another-domain.com

Destinataire externe

Si le serveur accepte → Open Relay, vulnérabilité critique permettant l’envoi massif de spam.

✉️ 6. Envoi manuel d’un email SMTP

HELO attacker.com
MAIL FROM:evil@attacker.com

RCPT TO:victim@domain.com

DATA
Subject: Test mail
Hello, this is a manual SMTP test.
.
QUIT

Permet d’envoyer un mail entièrement à la main (utile en pentest)

🕵️ 7. Spoofing d’adresse email

MAIL FROM:ceo@company.com

SMTP ne vérifie pas toujours l’expéditeur (sauf SPF/DMARC/DKIM)

Si accepté → spoofing possible

🧨 8. Brute-force d’identifiants SMTP

hydra -l admin -P rockyou.txt smtp://<IP> -s 587 -S

Bruteforce d’un compte SMTP avec STARTTLS

hydra -L users.txt -P pass.txt <IP> smtp

Bruteforce basique SMTP sur port 25

🔐 9. Test d’authentification SMTP

AUTH LOGIN

Lance l’auth en base64

echo -n "admin" | base64

Encode username

echo -n "password123" | base64

Encode mot de passe

🐞 10. Vulnérabilités SMTP fréquentes
🚨 Open Relay

Permet d’envoyer des mails via le serveur → très critique.

🔓 No Auth sur ports 587 / 465

Permet l’envoi de mails sans authentification.

🧨 Désactivation de STARTTLS

Communications en clair → interception possible.

🫥 Pas de SPF / DKIM / DMARC

Facilite le spoofing et l’usurpation d’adresse.

🛠 11. Outils SMTP utiles

swaks

Outil complet pour tester SMTP/SMTPS/STARTTLS

smtp-user-enum

Énumération d’utilisateurs automatique

hydra

Bruteforce d’identifiants SMTP

sendemail

Envoi d’e-mails scriptés (test de spoofing)

openssl s_client

Analyse TLS/SMTPS

🧹 12. Nettoyage

history -c

Efface les commandes SMTP effectuées en CLI

unset MAIL

Nettoyage des variables liées à l’envoi de mails

📚 13. Ressources utiles

RFC 5321 (SMTP)

Standard SMTP complet

Email Spoofing Cheatsheet

Méthodes de spoofing/défense

SPF / DKIM / DMARC Guides

Sécurisation contre le spoofing
