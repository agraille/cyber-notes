🧠 Radare2 – Cheatsheet Cybersécurité

Radare2 est un framework puissant pour reverse engineering, analyse binaire, debug, extraction, patching, et forensics.

🚀 1. Lancement & Chargement du binaire

r2 <fichier>

Ouvre un binaire dans Radare2

r2 -A <fichier>

Analyse automatique complète (recommandé au début)

r2 -d <fichier>

Ouvre un binaire en mode debug

🔍 2. Analyse du binaire

aa

Analyse automatique basique

aaa

Analyse approfondie (fonctions + CFG + imports)

aab

Analyse basique + binding (bons pour ELF/PE)

afl

Liste toutes les fonctions trouvées

aflj

Liste des fonctions au format JSON

agf

Affiche le graphe de la fonction courante

📁 3. Navigation dans le binaire

s main

Se déplacer à la fonction main

s sym.func

Aller à la fonction nommée

s entry0

Aller au point d’entrée du programme

s +10

Déplacer le curseur 10 octets en avant

🧭 4. Affichage & Désassemblage

pd 10

Désassembler 10 instructions

pdf

Désassembler toute la fonction courante

pdd

Désassemblage plus lisible (mode “demangled”)

px 32

Affiche 32 octets en hexadécimal

pdj 5

Désassemblage format JSON (utile pour scripts)

🔎 5. Recherche dans le binaire

/ str <texte>

Recherche une string dans le binaire

/ r /bin/sh

Recherche brute d'octets (ASCII)

/x 90 90

Recherche d'opcodes (ici : NOP NOP)

/c hello

Recherche d'une chaîne dans la section strings

iz

Liste toutes les strings trouvées

🧪 6. Informations sur le binaire

i

Infos globales du binaire

ie

Infos sur les entrées (entrypoints)

ii

Imports

iij

Imports en JSON

il

Liste des sections

ia

Architecture + bits (ex: x86, 64bit)

🐞 7. Debugging (gdb-like)

dc

Continue l’exécution

ds

Step dans l’instruction

dso

Step over

db <address>

Place un breakpoint

dbi

Liste les breakpoints

dr

Affiche les registres

dr eax

Affiche la valeur d’un registre spécifique

🧬 8. Patching & Écriture

wa nop

Écrit un NOP à l’emplacement courant

wa mov eax,1

Assemble et écrit l’instruction directement

wx 909090

Écrit des octets hexadécimaux (patching brut)

w < file

Écrit un fichier dans le binaire (overwrite)

wtf file.bin

Écrit le contenu du fichier à l’offset courant

🔐 9. Extraction & Dumping

pD 64 > dump.s

Dump de 64 instructions

px > dump.hex

Dump hexadécimal dans un fichier

wtf output.bin

Dump binaire à partir d'une position

📦 10. Modes d’affichage

V

Mode visuel (comme vim)

VV

Mode visuel Graph (CFG)

V!

Mode visualisation ASM (colorisé)

:hex

Vue hexadécimale

:pdf

Vue désassemblée de la fonction

🧮 11. Analyse cryptographique

yara rule.yar

Applique une règle YARA au binaire

r2 -q -c "px" /dev/mem

Analyse mémoire brute

pc

Détection automatique de constantes cryptographiques

📚 12. Radare2 dans des scripts (headless)

r2 -qc "aaa; afl; pdf @ main" <fichier>

Exécuter Radare2, lancer commandes, quitter

r2pipe (Python)

Automatise analyses via scripts Python

🧹 13. Sortie & Nettoyage

q

Quitter Radare2

q!

Quitter sans sauvegarder

📚 14. Ressources utiles

Radare2 Book

Documentation officielle complète

r2wiki

Référence des commandes

Cutter (GUI)

Interface graphique pour Radare2

radare2-extras

Plugins supplémentaires
