# 🛠️ Ghidra - Cheatsheet Cyber Sécurité

Ce fichier regroupe les **étapes clés** et **astuces** pour utiliser **Ghidra**, un framework d’ingénierie inverse développé par la NSA, idéal pour l’analyse de binaires dans un contexte de cybersécurité.

---

## 1️⃣ Importer un binaire

File > Import File  
> Sélectionne le fichier binaire à analyser (EXE, ELF, DLL, etc.).

Choisir le format si nécessaire, cliquer sur **OK**.

---

## 2️⃣ Créer un projet

File > New Project  
> Crée un nouveau projet (choisir Standalone Project).

Nommer et sauvegarder le projet.

---

## 3️⃣ Analyse automatique

Après l’import, Ghidra propose d’analyser le fichier.  
> Cocher les options recommandées et lancer l’analyse.

---

## 4️⃣ Navigation dans le binaire

- **Code Browser** : fenêtre principale pour parcourir le code, fonctions, variables.  
- **Symbol Tree** : liste des fonctions, labels, variables globales.  
- **Listing** : affiche le désassemblage et le code désassemblé.  
- **Decompile Window** : vue en pseudo-code C pour faciliter la compréhension.

---

## 5️⃣ Rechercher des fonctions vulnérables

- **Chercher des fonctions dangereuses classiques** :  
  strcpy, strncpy, strcat, strncat, sprintf, vsprintf, gets, scanf, sscanf, memcpy, memmove, atoi, system, exec*, fopen, fread, fwrite, read, write  
> Ces fonctions sont souvent liées à des vulnérabilités telles que buffer overflow, format string, injection de commande, etc.

- **Identifier les entrées utilisateur** :  
  Rechercher les fonctions qui récupèrent des données externes (ex : read, recv, fgets) pour tracer la source des données.

- **Rechercher les chaînes de caractères** :  
  Search > For Strings  
> Permet de trouver des indices sur les inputs, messages d’erreur, ou commandes utilisées dans le binaire.

- **Vérifier les validations d’entrée** :  
  Analyser les conditions sur les tailles, boucles, comparaisons avant l’utilisation des fonctions sensibles.

- **Utiliser les plugins et scripts** :  
  Certains scripts Ghidra peuvent automatiser la recherche de fonctions vulnérables.

---

## 6️⃣ Renommer et annoter

- Cliquer droit sur une fonction ou variable > Rename  
> Facilite la lecture.

- Ajouter des commentaires (Right-click > Comment)  
> Documenter les parties critiques.

---

## 7️⃣ Debug et scripts

- Utiliser le debugger intégré pour analyser l’exécution (nécessite configuration).  
- Scripts Python ou Java via Window > Script Manager pour automatiser des tâches.

---

## 8️⃣ Exporter et sauvegarder

File > Export Program  
> Exporter en différents formats.

Sauvegarder régulièrement le projet.

---

## 📌 Ressources utiles

- Documentation officielle : https://ghidra-sre.org/  
- Tutoriels vidéo : https://www.youtube.com/c/GhidraTutorials  
- Scripts et plugins : https://github.com/NationalSecurityAgency/ghidra/wiki/Using-Ghidra  
- Communauté & exemples : https://reverseengineering.stackexchange.com/questions/tagged/ghidra
