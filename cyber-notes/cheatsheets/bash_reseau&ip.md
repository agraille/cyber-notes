# 🐚 Bash Cheatsheet – Réseau & IP

---

## 🌐 Commandes réseau

### 🔍 Affichage d'adresses IP

ip a
> Affiche toutes les interfaces réseau et leurs adresses IP.  

hostname -I
> Affiche l’adresse IP locale.  

curl ifconfig.me 
> Affiche ton IP publique (via HTTP).  

---

### 🔧 Informations sur les interfaces

ifconfig  
> Affiche les interfaces réseau (nécessite net-tools).  

---

### 🌍 DNS et nom d’hôte

hostname  
> Affiche le nom d’hôte actuel.  

nslookup google.com  
> Résout le nom DNS d’un domaine (IP).  

dig google.com  
> Donne des infos détaillées sur la résolution DNS.  

host google.com  
> Donne une résolution DNS simple.

---

## 🔌 Connexions réseau

ping "ip"
> Vérifie si une machine répond.  

ping -c 4 "ip"
> Ping Google 4 fois.

ss -tulnp  
> Affiche les sockets TCP/UDP ouverts avec PID.  

lsof -i  
> Liste tous les fichiers liés à des connexions réseau.  

lsof -i :80  
> Filtre uniquement le port 80 (HTTP).  

lsof -iTCP -sTCP:LISTEN -P  
> Liste les processus écoutant sur les ports TCP.

---

## 📡 Découverte réseau locale

arp -a  
> Affiche la table ARP (machines connues sur le réseau local).  

ip neigh  
> Affiche les voisins (cache ARP).  

nmap -sn 192.168.1.0/24  
> Scan ping sur tout un réseau local.  

---

## 🧰 Outils et diagnostics réseau

traceroute google.com  
> Montre le chemin pris par les paquets vers un hôte.  

tracepath google.com  
> Comme traceroute mais sans nécessiter les privilèges root. 

mtr google.com  
> Combine traceroute et ping en continu.  

whois google.com  
> Donne les infos d’enregistrement du domaine.  

dig +short myip.opendns.com @resolver1.opendns.com  
> Donne ton IP publique via OpenDNS (rapide et précis).

---

## 🛠️ Alias utiles à mettre dans ~/.bashrc
```
alias ll='ls -la'  
```
> Affiche tous les fichiers avec détails.  

alias iplocal='ip a | grep inet'  
> Récupère rapidement les adresses IP.  

alias ippublique='curl ifconfig.me'  
> Alias pour connaître ton IP publique.  

alias ports='ss -tulnp'  
> Voir rapidement les ports ouverts.  

alias interfaces='ip link show'  
> Voir les interfaces réseau disponibles.
