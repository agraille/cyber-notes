# 🐞 GDB - Cheatsheet Cybersécurité

Ce fichier fournit les commandes essentielles GDB pour analyser, débugger et exploiter des binaires sous Linux, particulièrement utile pour les CTFs, les failles de type buffer overflow ou les binaires protégés.

---

## ⚙️ 1. Lancer GDB

gdb ./binaire  
> Ouvre GDB sur un binaire local

gdb -q ./binaire  
> Mode silencieux (sans bannière)

gdb -p PID  
> Attache GDB à un processus en cours

---

## 🏁 2. Exécution

run ARG1 ARG2  
> Lance le binaire avec des arguments

start  
> Lance l'exécution et arrête à main()

continue (ou c)  
> Reprend l'exécution après un break

next (ou n)  
> Exécute la ligne suivante (sans entrer dans les fonctions)

step (ou s)  
> Exécute la ligne suivante (entre dans les fonctions)

finish  
> Continue jusqu’à la fin de la fonction actuelle

---

## 🎯 3. Breakpoints

break *0xADDRESS  
> Met un breakpoint à une adresse mémoire

break fonction  
> Breakpoint à l'entrée d'une fonction

break ligne  
> Breakpoint à une ligne du code

info breakpoints  
> Liste des breakpoints

delete NUM  
> Supprime un breakpoint

disable/enable NUM  
> Désactive/réactive un breakpoint

---

## 🧠 4. Inspection mémoire & registres

info registers  
> Affiche les registres

x/32x $esp  
> Affiche 32 cases mémoire en hex à partir de ESP

x/s $esp  
> Affiche la chaîne de caractères pointée par ESP

x/20i $eip  
> Affiche les instructions (désassemblage)

x/10xw 0xADDR  
> Affiche 10 words en hex depuis une adresse

display $eax  
> Affiche en permanence la valeur du registre EAX

---

## 🧪 5. Fuzzing / Payloads

set args $(python3 -c "print('A'*100)")  
> Injecte une payload dans les arguments

run  
> Lance le binaire avec l'input contrôlé

set $eip = 0xADDR  
> Écrase le registre EIP (test d'overflow)

---

## 🧰 6. Outils GDB utiles

### GEF (GDB Enhanced Features)

Installation :

echo "source ~/gef.py" >> ~/.gdbinit  
wget -O ~/gef.py https://gef.blah.cat/gef.py  

Commandes utiles (avec GEF) :

context  
> Affiche registre + stack + code

pattern create 100  
> Crée un motif pour trouver offset

pattern search 0x41414141  
> Cherche un motif dans la stack/heap

pattern offset 0x41414141  
> Calcule l’offset exact d’un overflow

### pwndbg

Alternative à GEF :  
https://github.com/pwndbg/pwndbg

---

## 🔓 7. Cas typique : Débogage d’un buffer overflow

1. Lancer GDB :  
gdb ./vuln

2. Définir les arguments :  
set args $(python3 -c "print('A'*200)")

3. Lancer et observer le crash :  
run  
info registers

4. Vérifier EIP / RIP :  
pattern create 200  
set args $(pattern_output)  
run  
pattern search $eip

5. Remplacer avec un shellcode ou jump :  
set $eip = 0xdeadbeef  
run

---

## 🧹 8. Divers

info functions  
> Liste toutes les fonctions

info files  
> Montre les segments du binaire (PIE, ASLR, etc.)

disassemble main  
> Désassemble la fonction main

vmmap  
> Affiche la mémoire (nécessite GEF/pwndbg)

---

## 📚 Ressources

- https://github.com/hugsy/gef  
- https://github.com/pwndbg/pwndbg  
- https://book.hacktricks.xyz/pentesting-hard/low-level-exploitation/basic-buffer-overflows  
- https://www.exploit-db.com/
