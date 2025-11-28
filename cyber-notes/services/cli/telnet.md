📟 TELNET – Cheatsheet Cybersécurité

Telnet est un protocole réseau utilisé pour connexion distante, debug, tests de services, impression, et analyse brute TCP, souvent utilisé en pentest pour tester des ports, banner grabbing, SMTP/POP/IMAP, ou contrôle legacy.

🚀 1. Connexion simple à un service

telnet <IP> <port>

Se connecte en Telnet brut sur n’importe quel port TCP (test de service)

telnet 192.168.1.10 23

Connexion Telnet classique (port 23)

🔍 2. Banner Grabbing

telnet <IP> 80

Connexion brute HTTP pour tester le serveur

GET / HTTP/1.1
Host: target.com

Permet de lire la bannière et tester un service web en manuel

📡 3. Test SMTP / POP / IMAP (manuellement)
📧 SMTP

telnet <IP> 25

Connexion SMTP non chiffrée

HELO test

Présentation au serveur

MAIL FROM:test@test.com

RCPT TO:user@domain.com

DATA
Test email
.

Envoi manuel d’e-mails pour test

📮 POP3

telnet <IP> 110

Connexion POP3

USER admin
PASS password
LIST
RETR 1

Lecture des mails en POP3

📨 IMAP

telnet <IP> 143

Connexion IMAP

a login admin pass123
a list "" *
a select INBOX
a fetch 1 all

Lecture d’emails via IMAP

🧪 4. Test de ports & services
🔎 Tester un port ouvert

telnet <IP> 21

Test FTP (port 21)

telnet <IP> 3306

Test MySQL (bannière si non sécurisé)

telnet <IP> 22

SSH refuse mais affiche souvent une bannière SSH

🧰 5. Utilisation en Debug réseau
Test d'un serveur web sur un chemin spécifique

telnet <IP> 80
GET /login HTTP/1.1
Host: <IP>

Permet de tester manuellement réponses HTTP, redirections, cookies…

Debug d'un serveur TCP custom

telnet <IP> 4444

Accès direct au service pour tester protocole perso

🧱 6. Exploitation basique via Telnet
🔓 Tentative de login simple (si service Telnet actif)

telnet <IP> 23
login: admin
password: admin

Souvent vulnérable en IoT / legacy

🐍 Injection simple (si protocole vulnérable)

telnet <IP> <port>
{"cmd":"whoami"}

Utile pour services TCP custom vulnérables

🛡 7. Désactivation de Telnet (défense)
Sur Linux :

systemctl stop telnet
systemctl disable telnet

Désactive le service

Sur Cisco / Routeurs :

no line vty 0 4
transport input ssh

Désactive Telnet, force SSH

🪓 8. Alternatives utiles à Telnet

nc <ip> <port>

Netcat → plus puissant pour tests TCP

nmap -p <port> <ip>

Détection de services

openssl s_client -connect <ip>:443

Connexion TLS (Telnet ne gère pas SSL)

curl --raw

Test HTTP brut

🧹 9. Nettoyage & Sécurité

history -c

Nettoie l’historique shell

logout / quit

Quitte proprement un service telnet ouvert

📚 10. Ressources utiles

RFC 854 – Telnet Protocol

Norme officielle Telnet

IANA Ports

Liste des ports TCP utilisables avec Telnet

OWASP Testing Guide – Network Services

Guide de tests de services réseau
