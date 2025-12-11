# 📡 Wireshark - Cheatsheet Cyber Sécurité

---

## 1️⃣ Capture de trafic réseau

- Lancer Wireshark et sélectionner l’interface réseau active (ex : eth0, wlan0).  

- Filtrer la capture directement via la barre de filtre (ex : ip.addr == 192.168.1.10).

---

## 2️⃣ Filtres d’affichage (Display Filters)

```
ip  
```
> Affiche tous les paquets IP

```
tcp.port == 80  
```
> Filtre le trafic TCP sur le port 80 (HTTP)

```
udp.port == 53  
```
> Filtre le trafic DNS (port UDP 53)

```
http.request.method == "GET"  
```
> Affiche les requêtes HTTP GET

```
dns.qry.name == "example.com"  
```
> Recherche les requêtes DNS pour example.com

```
tcp.flags.syn == 1 and tcp.flags.ack == 0  
```
> Filtre les paquets SYN pour détecter les débuts de connexions TCP

```
frame contains "password"  
```
> Recherche les paquets contenant le mot "password"

---

## 3️⃣ Analyse approfondie

```
- Utiliser “Follow TCP Stream” (clic droit sur un paquet TCP)  
```
> Affiche la session complète en texte clair (utile pour HTTP, FTP, etc.).

```
- Statistiques > Protocol Hierarchy  
```
> Vue d’ensemble des protocoles capturés.

```
- Statistiques > Conversations  
```
> Analyse des échanges entre adresses IP et ports.

```
- Statistiques > Endpoints  
```
> Liste des IP impliquées dans la capture.

---

## 4️⃣ Exporter des données

```
- File > Export Specified Packets  
```
> Exporter les paquets sélectionnés ou filtrés au format pcap.

- Exporter les données brutes d’un flux TCP/UDP (Follow TCP Stream > Save As).

---

## 5️⃣ Astuces pour la cybersécurité

- **Détection d’attaques** : filtrer les paquets SYN flood avec  
  tcp.flags.syn == 1 and tcp.flags.ack == 0 and frame.len > 1000

- **Extraction de fichiers** : extraire des fichiers transférés via HTTP, FTP, SMB, etc.

- **Détection de mots de passe en clair** : chercher “password”, “Authorization”, “Basic” dans les flux HTTP.

- **Suivi de sessions chiffrées** : analyser le TLS handshake et certificats.

- **Utiliser Tshark** (version en ligne de commande de Wireshark) pour automatiser des analyses :  
  tshark -i eth0 -w capture.pcap

---

## 6️⃣ Commandes utiles Tshark

```
tshark -i eth0  
```
> Capture sur interface eth0

```
tshark -r fichier.pcap -Y "http.request.method == GET"  
```
> Analyse un fichier pcap et filtre les requêtes HTTP GET

```
tshark -i eth0 -f "tcp port 80" -c 100  
```
> Capture 100 paquets sur port 80 uniquement

---

## 📌 Ressources utiles

- Documentation officielle : https://www.wireshark.org/docs/  
- Wireshark Cheat Sheet : https://www.wireshark.org/docs/wsug_html_chunked/ChapterIntroduction.html  
- Tutoriels vidéo : https://www.youtube.com/user/wiresharkorg  
- Tshark man page : https://www.wireshark.org/docs/man-pages/tshark.html
