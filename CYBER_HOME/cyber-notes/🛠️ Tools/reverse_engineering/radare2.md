# ⚡ Radare2 - Cheatsheet Cyber Sécurité

Guide orienté **reverse engineering** et **exploitation de binaires** avec Radare2. Focus sur l'analyse rapide, le pwn et l'automatisation.

---

## 📖 Qu'est-ce que Radare2 ?

**Radare2 (r2)** est un framework open-source de reverse engineering en ligne de commande. Il permet de :
- Désassembler et analyser des binaires
- Déboguer des programmes
- Patch de binaires
- Exploitation de vulnérabilités
- Analyse de firmware
- Scripting et automatisation

**Points forts** :
- Ultra-léger et rapide
- Support de 40+ architectures
- Interface en ligne de commande puissante
- Scriptable (r2pipe)

---

## 1️⃣ Commandes de Base

### Lancement

```bash
# Ouvrir un binaire
r2 ./binary

# Analyse automatique
r2 -A ./binary          # -A = aaa (analyse complète)
r2 -AA ./binary         # -AA = aaaa (analyse très agressive)

# Mode debug
r2 -d ./binary
r2 -d ./binary arg1 arg2

# Mode write (permet modification)
r2 -w ./binary

# Sans analyse
r2 -n ./binary

# Quiet mode
r2 -q ./binary
```

### Navigation

```r2
# Aller à une adresse
s main                  # Seek to main
s 0x08048484            # Seek to address
s entry0                # Seek to entry point
s sym.vulnerable        # Seek to symbol

# Navigation relative
s+ 10                   # Avancer de 10 bytes
s- 10                   # Reculer de 10 bytes

# Historique
s-                      # Adresse précédente
s+                      # Adresse suivante

# Info position actuelle
s                       # Afficher adresse actuelle
```

---

## 2️⃣ Analyse

### Analyse automatique

```r2
# Analyses progressives
aa                      # Analyse basique
aaa                     # Analyse complète (recommandé)
aaaa                    # Analyse très agressive
aac                     # Analyse des appels
aab                     # Analyse des blocs
aan                     # Analyse et renommage auto
aar                     # Analyse des références
aas                     # Analyse des strings
```

### Informations

```r2
# Info binaire
i                       # Info générale
ii                      # Imports
ie                      # Entry points
iS                      # Sections
iz                      # Strings dans .rodata
izz                     # Toutes les strings
il                      # Libraries
is                      # Symbols

# Checksec
iI                      # Info détaillée (avec protections)
```

---

## 3️⃣ Fonctions

### Analyse de fonctions

```r2
# Lister les fonctions
afl                     # List all functions
afll                    # Long listing avec détails

# Info sur fonction actuelle
afi                     # Function info
afv                     # Variables locales
afa                     # Arguments

# Analyse de fonction spécifique
af @ sym.main           # Analyze function at main
af @ 0x08048484         # Analyze function at address
```

### Navigation dans les fonctions

```r2
# Aller à une fonction
s sym.main
s sym.vulnerable

# Xrefs (références croisées)
axt @ sym.main          # Xrefs TO main
axf @ sym.main          # Xrefs FROM main
```

---

## 4️⃣ Désassemblage

### Désassembler

```r2
# Désassembler
pdf                     # Print disassembly function
pdf @ sym.main          # Disas main
pd 20                   # Disas 20 instructions
pd -20                  # Disas 20 instructions avant
pD 100                  # Disas 100 bytes

# Avec opcodes
pdo                     # Disas avec opcodes

# Graph mode
VV                      # Visual mode graph
V                       # Visual mode
```

### Mode visuel

```r2
V                       # Entrer en mode visuel
VV                      # Visual graph mode

Navigation en mode visuel :
j/k         : Haut/Bas
h/l         : Gauche/Droite
p/P         : Changer de vue
:           : Mode commande
q           : Quitter

Dans VV (graph) :
Space       : Changer le layout
R           : Randomize colors
+/-         : Zoom
Tab         : Passer au bloc suivant
```

---

## 5️⃣ Mémoire et Recherche

### Examiner la mémoire

```r2
# Print memory
px 100                  # Hexdump 100 bytes
px @ 0x08048000
pxw 40                  # 40 words
pxq 20                  # 20 qwords (x64)

# Print as
ps                      # Print string
pS                      # Print Pascal string
pc                      # Print C array
pj                      # Print JSON
```

### Recherche

```r2
# Chercher des strings
/ password              # Chercher "password"
/i password             # Case insensitive
/e /bin/sh              # Chercher /bin/sh

# Chercher en hexa
/x 4141414241           # Chercher AAAAB

# Chercher des patterns
/R pop rdi              # Chercher gadgets ROP
/R pop rsi
/R ret

# Chercher dans toutes les fonctions
/af vulnerable          # Chercher fonction contenant "vulnerable"
```

---

## 6️⃣ Debugging

### Breakpoints

```r2
# Mettre des breakpoints
db sym.main             # Break at main
db 0x08048484           # Break at address
dbt                     # Tracer les calls

# Lister les breakpoints
dbl

# Supprimer
db- sym.main            # Remove breakpoint
db-*                    # Remove all
```

### Exécution

```r2
# Lancer le programme
dc                      # Continue
dcu sym.main            # Continue until main
ds                      # Step
dso                     # Step over
dss                     # Skip instruction

# Etat du programme
dr                      # Afficher registres
dr?                     # Aide registres
dr eax                  # Afficher eax
dr eax=0x41414141       # Modifier eax
```

### Pile et registres

```r2
# Registres
dr                      # Tous les registres
drr                     # Affichage détaillé
dr=                     # Affichage visuel

# Stack
pxw 40 @ esp            # 40 words depuis ESP
pxr @ esp               # Stack avec références

# Backtrace
dbt                     # Backtrace
```

---

## 7️⃣ Exploitation

### Détection de vulnérabilités

**Buffer Overflow**
```r2
# Chercher fonctions dangereuses
afl | grep gets
afl | grep strcpy
afl | grep sprintf

# Analyser la fonction
s sym.vulnerable
pdf
afi                     # Info sur taille buffer
```

**Pattern De Bruijn**
```r2
# Générer pattern
ragg2 -P 200 -r        # Pattern de 200 bytes

# Ou avec wopO
wopO                    # Generate pattern
wopO 200                # Pattern de 200 bytes

# Trouver l'offset
wopO `dr eip`          # Offset depuis EIP écrasé
```

**ROPgadget**
```r2
# Chercher gadgets
/R pop rdi; ret
/R pop rsi; pop r15; ret
/R pop rdx; ret
/R syscall
/R int 0x80

# Lister tous les gadgets
"/R/"                   # Liste interactive
```

### Exploitation pratique

**ret2libc**
```r2
# 1. Trouver system()
afl | grep system
s sym.imp.system
s

# 2. Trouver /bin/sh
iz | grep /bin/sh
/ /bin/sh

# 3. Trouver gadget pop rdi
/R pop rdi

# 4. Construire payload
# offset + pop_rdi + binsh_addr + system_addr
```

---

## 8️⃣ Protections Binaires

### Checksec

```r2
iI                      # Info détaillée

# Vérifier :
# NX : No-eXecute
# PIE : Position Independent
# Canary : Stack canary
# RELRO : GOT protection
# RPATH/RUNPATH
```

**Analyse détaillée**
```r2
# Canary
afl | grep stack_chk    # Chercher stack_chk_fail
pdf @ sym.main | grep fs:0x28  # Vérifier canary check

# NX
iS | grep -e stack -e heap  # Permissions des sections

# RELRO
ii | grep BIND_NOW      # Full RELRO si présent
```

---

## 9️⃣ Patch Binaire

### Modification en mémoire

```r2
# Mode write
r2 -w ./binary

# Patcher des bytes
wx 9090 @ 0x08048484    # Écrire NOP (0x90)
wx c3 @ 0x08048500      # Écrire RET (0xc3)

# Patcher une instruction
wa jmp 0x08048600       # Write assembly
wa nop
wa ret

# Patch de string
w "PATCHED" @ 0x08048000

# Sauvegarder
wc                      # Commit changes
```

### Backup et restore

```r2
# Sauvegarder avant modifications
wts backup.r2           # Take snapshot

# Restaurer
wfs backup.r2           # From snapshot
```

---

## 🔟 Scripts et Automatisation

### r2pipe (Python)

```python
import r2pipe

# Ouvrir le binaire
r2 = r2pipe.open("./binary")

# Analyse
r2.cmd("aaa")

# Lister les fonctions
functions = r2.cmdj("aflj")  # j = JSON output
for func in functions:
    print(f"{func['name']}: {hex(func['offset'])}")

# Chercher strings
strings = r2.cmdj("izzj")
for s in strings:
    if "password" in s['string']:
        print(f"Found: {s['string']} at {hex(s['vaddr'])}")

# Désassembler
disas = r2.cmd("pdf @ sym.main")
print(disas)

# Fermer
r2.quit()
```

### Scripts r2

**Script 1 : Trouver toutes les vulnérabilités**
```r2
# vuln_finder.r2
aaa
echo "=== Dangerous Functions ==="
afl | grep gets
afl | grep strcpy
afl | grep sprintf
afl | grep scanf

echo "=== Interesting Strings ==="
izz | grep password
izz | grep admin
izz | grep key

echo "=== Potential ROP Gadgets ==="
/R pop rdi
/R pop rsi
/R syscall
```

**Utilisation**
```bash
r2 -q -c ". vuln_finder.r2" ./binary
```

### Automatisation complète

```python
# exploit_finder.py
import r2pipe
import sys

def find_vulnerabilities(binary_path):
    r2 = r2pipe.open(binary_path)
    r2.cmd("aaa")
    
    print("[+] Analyzing", binary_path)
    
    # Checksec
    info = r2.cmdj("iIj")
    print(f"NX: {info['nx']}")
    print(f"PIE: {info['pic']}")
    print(f"Canary: {info['canary']}")
    
    # Dangerous functions
    dangerous = ["gets", "strcpy", "sprintf", "scanf"]
    print("\n[+] Dangerous functions:")
    for func in dangerous:
        result = r2.cmd(f"afl | grep {func}")
        if result:
            print(f"  - {func}: FOUND")
    
    # Strings
    print("\n[+] Interesting strings:")
    strings = r2.cmdj("izzj")
    keywords = ["password", "admin", "key", "flag"]
    for s in strings:
        for kw in keywords:
            if kw in s['string'].lower():
                print(f"  - {s['string']} at {hex(s['vaddr'])}")
    
    r2.quit()

if __name__ == "__main__":
    find_vulnerabilities(sys.argv[1])
```

---

## 1️⃣1️⃣ Analyse de Malware

### Workflow d'analyse

```r2
# 1. Info basique
r2 -AA malware.exe
iI                      # Protections
iS                      # Sections
ii                      # Imports

# 2. Strings suspectes
izz | grep -i http
izz | grep -i cmd
izz | grep -i password
izz | grep -i registry

# 3. API Windows suspectes
afl | grep CreateProcess
afl | grep VirtualAlloc
afl | grep WriteProcessMemory
afl | grep LoadLibrary

# 4. Entry point
ie
s entry0
pdf
```

### Extraction de strings

```r2
# Toutes les strings
izz > strings.txt

# Filtrer
izz | grep http:// > urls.txt
izz | grep -E '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' > ips.txt

# Strings encodées
/x 687474703a2f2f      # Chercher "http://" en hex
```

### Analyse de packer

```r2
# Identifier les sections
iS

# Section suspecte ?
# .text très petite + .data grande = packed

# Chercher le stub de déchiffrement
s entry0
pdf

# Identifier boucle de XOR
/c xor
/c rol
```

---

## 1️⃣2️⃣ Analyse de Firmware

### Import de firmware

```bash
# Ouvrir raw binary
r2 -a arm -b 32 firmware.bin   # ARM 32-bit
r2 -a mips -b 32 firmware.bin  # MIPS 32-bit
```

### Analyse

```r2
# Chercher l'entry point
/x 00 00 a0 e3          # ARM : mov r0, #0

# Analyser
aaa

# Chercher des strings
izz | grep -i model
izz | grep -i version
izz | grep -i password
```

---

## 1️⃣3️⃣ Intégrations

### r2ghidra (Décompilateur)

```bash
# Installation
r2pm -ci r2ghidra

# Utilisation
r2 -A ./binary
s sym.main
pdg                     # Print decompiled (Ghidra)
```

### r2frida

```bash
# Installation
r2pm -ci r2frida

# Attacher à un processus
r2 frida://0            # Liste les processus
r2 frida://1234         # Attacher au PID 1234
r2 frida://spawn/app    # Spawn et attacher
```

---

## 1️⃣4️⃣ Techniques Avancées

### Emulation avec ESIL

```r2
# ESIL = Evaluable Strings Intermediate Language

# Activer ESIL
e asm.emu=true          # Enable emulation
aeim                    # Initialize ESIL VM

# Émuler des instructions
aes                     # Emulate step
aec                     # Continue emulation
aer                     # ESIL registers
```

### Analyse de structures

```r2
# Définir une structure
"td struct User { char name[32]; int age; };"

# Appliquer
pf.User @ 0x08048000

# Afficher
pf @ 0x08048000
```

### Graphes

```r2
# Call graph
agc                     # Call graph
agC                     # Call graph (reversed)

# Data references
agd                     # Data references
agD                     # Data references (reversed)

# Export DOT
ag > graph.dot
dot -Tpng graph.dot > graph.png
```

---

## 1️⃣5️⃣ Commandes Utiles

### Flags et bookmarks

```r2
# Créer un flag
f flag_name @ 0x08048484

# Lister les flags
f

# Supprimer
f-flag_name

# Renommer
fr old_name new_name
```

### Commentaires

```r2
# Ajouter commentaire
CCu "This is vulnerable" @ 0x08048484

# Afficher
CC @ 0x08048484

# Supprimer
CC- @ 0x08048484
```

### Project management

```r2
# Sauvegarder projet
Ps project_name

# Ouvrir projet
Po project_name

# Lister projets
P
```

---

## 1️⃣6️⃣ Cheatsheet Rapide

### Navigation
```
s main                  # Seek to main
s 0x08048484            # Seek to address
s+ 10 / s- 10           # Relative seek
s- / s+                 # History
```

### Analyse
```
aaa                     # Analyze
afl                     # List functions
pdf                     # Print disasm function
VV                      # Visual graph
```

### Recherche
```
/ string                # Search
/x 414141               # Search hex
/R pop rdi              # ROP gadgets
iz / izz                # Strings
```

### Debug
```
db sym.main             # Breakpoint
dc                      # Continue
ds                      # Step
dr                      # Registers
px 40 @ esp             # Examine stack
```

### Exploitation
```
wopO 200                # Pattern
wopO `dr eip`           # Offset
/R pop rdi              # ROP
ii                      # Imports (ret2libc)
```

### Info
```
i                       # Info
iI                      # Checksec
ii                      # Imports
iS                      # Sections
iz                      # Strings
```

---

## 1️⃣7️⃣ Raccourcis Essentiels

```
?                       # Aide générale
??                      # Aide courte
?*                      # Aide en mode r2 cmd
cmd?                    # Aide sur commande

~ grep                  # Pipe grep
~...                    # Less
| shell cmd             # Pipe to shell

@@                      # Iterator
@@ function             # Pour chaque fonction
@@f                     # Pour chaque fonction
```

---

## 1️⃣8️⃣ Configuration

### .radare2rc

```bash
# ~/.radare2rc
e asm.syntax=intel
e scr.utf8=true
e scr.color=3
e asm.describe=true
e asm.emu=true
```

### Thèmes

```r2
eco                     # Liste themes
eco matrix              # Theme matrix
eco dracula             # Theme dracula
```

---

## 1️⃣9️⃣ Ressources

### Documentation
- Radare2 Book : https://book.rada.re/
- GitHub : https://github.com/radareorg/radare2
- Wiki : https://r2wiki.readthedocs.io/

### Plugins
- r2ghidra : Décompilateur Ghidra
- r2frida : Dynamic instrumentation
- r2dec : Décompilateur
- r2pm : Package manager

### Communauté
- Telegram : @radare
- IRC : #radare on irc.freenode.net
- Twitter : @radareorg

### Labs
- Crackmes.one
- pwnable.kr avec r2
- CTF challenges

---

## 💡 Tips Pro

1. **Toujours analyser avec aaa** avant exploration
2. **Mode visuel VV** pour comprendre le flux
3. **r2pipe pour automation** en Python
4. **wopO pour patterns** De Bruijn
5. **pdg (r2ghidra)** pour décompilation rapide
6. **/R pour ROP gadgets** instant
7. **Sauvegarder en projet** pour gros binaires
8. **ESIL pour émulation** sans exécution
9. **Pipe avec ~** pour filtrer (grep-like)
10. **r2frida pour analyse dynamique** mobile/desktop

---

**⚡ Radare2 est l'outil de RE le plus puissant en CLI. Rapide, léger et scriptable !**
