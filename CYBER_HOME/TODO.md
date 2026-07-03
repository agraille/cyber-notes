voir le fonctionnement exact de newgrp pourquoi possibilite de passer sur le group docker sur le challenge kobold
view-source:URL

voir TNS
Résumé de la section : Oracle TNS

Concepts Clés :

Oracle TNS (Transparent Network Substrate) : C'est un protocole de communication qui permet aux applications clientes de se connecter et de communiquer avec les bases de données Oracle sur un réseau. Il gère des aspects cruciaux comme la résolution de noms, la gestion des connexions et la sécurité.
SID (System Identifier) : Un nom unique qui identifie une instance de base de données Oracle spécifique sur un serveur. Les clients doivent spécifier le bon SID pour se connecter à la base de données souhaitée.
Fichiers de Configuration : Les deux fichiers de configuration principaux sont tnsnames.ora (côté client), qui mappe les noms de service aux adresses réseau, et listener.ora (côté serveur), qui configure le processus d'écoute qui reçoit les connexions entrantes.
Application Pratique :
La section montre comment énumérer et interagir avec un service TNS. Le processus commence par un scan de port (nmap) pour trouver le port par défaut 1521/tcp. Ensuite, des outils comme nmap ou ODAT (Oracle Database Attacking Tool) sont utilisés pour découvrir les SIDs valides et tenter de deviner les mots de passe. Une fois les informations d'identification obtenues, l'outil sqlplus est utilisé pour se connecter à la base de données, énumérer des informations sensibles comme les tables et les privilèges, et potentiellement escalader les privilèges en se connectant en tant que sysdba.

Points de Vigilance :

Les configurations par défaut représentent souvent un risque. Faites attention au port 1521, aux SIDs courants comme XE, et aux identifiants par défaut comme scott/tiger.
Même un utilisateur avec des privilèges faibles peut parfois les élever de manière significative en se connectant as sysdba, ce qui lui donne un contrôle administratif complet sur la base de données.
