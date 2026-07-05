# Active Directory Defense

## 1. Le concept

### Où ça se situe dans NIST

L'Active Directory est un système d'identité et de gestion d'accès : sa défense s'inscrit donc naturellement dans **PR.AC (Identity Management, Authentication and Access Control)** de la fonction Protect, complétée par **DE.CM (Continuous Monitoring)** pour la détection des abus. Le NIST insiste sur le fait que la gestion des identités est un contrôle central : compromettre l'identité (un compte, un ticket d'authentification) revient souvent à contourner tous les autres contrôles de sécurité, car le système fait alors confiance à l'attaquant comme s'il était un utilisateur légitime.

### C'est quoi concrètement

L'AD est la cible numéro un en environnement d'entreprise, car sa compromission donne accès à l'ensemble du système d'information. La défense de l'AD repose sur trois piliers : détecter les techniques d'attaque connues qui ciblent le protocole d'authentification Kerberos, limiter la propagation d'une compromission grâce à un modèle de cloisonnement des privilèges, et surveiller les événements qui trahissent un abus du protocole.

## 2. Pourquoi ça marche (le mécanisme)

Kerberos fonctionne par délivrance de "tickets" : un ticket prouve qu'un utilisateur ou un service a le droit d'accéder à une ressource, sans avoir à retransmettre le mot de passe à chaque fois. Plusieurs attaques exploitent des propriétés de ce mécanisme :

- **Kerberoasting** : n'importe quel utilisateur du domaine peut légitimement demander un ticket de service (TGS) pour un compte associé à un SPN (Service Principal Name). Ce ticket est chiffré avec un dérivé du mot de passe du compte de service. Un attaquant peut donc demander ce ticket, puis tenter de casser le mot de passe hors ligne, sans jamais déclencher d'échec d'authentification visible.
- **Golden Ticket** : le compte `krbtgt` signe tous les tickets d'authentification (TGT) du domaine. Si son hash est volé, un attaquant peut forger lui-même des TGT valides pour n'importe quel compte, sans jamais interroger le contrôleur de domaine — d'où la difficulté de détection.
- **Silver Ticket** : variante où l'attaquant forge directement un ticket de service (TGS) en utilisant le hash du compte de service ciblé, sans passer par le contrôleur de domaine à aucun moment.
- **DCSync** : un contrôleur de domaine dispose nativement du droit de répliquer les données d'identité (dont les hashs de mots de passe) depuis un autre contrôleur. Un attaquant qui obtient ce droit sur un compte ordinaire peut se faire passer pour un contrôleur de domaine et extraire tous les hashs du domaine en une seule requête.

Ce qui rend ces attaques dangereuses, c'est qu'elles abusent de fonctionnalités légitimes du protocole plutôt que d'exploiter une faille logicielle — la détection doit donc se faire sur des **anomalies de comportement**, pas sur une signature d'exploit.

## 3. Mise en œuvre — le chemin concret

### Détecter le Kerberoasting

Un volume anormal de demandes de tickets de service chiffrés avec un algorithme faible (RC4) est un signal fort :
```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4769} |
  Where-Object { $_.Message -match 'Ticket Encryption Type:\s+0x17' } |
  Select-Object TimeCreated, @{n='Account';e={($_.Message -split "Account Name:\s+")[1] -split "`n" | Select -First 1}}
```

Mitigation : mots de passe de comptes de service longs (25+ caractères), ou passage à des comptes de service gérés (gMSA) dont le mot de passe est généré et changé automatiquement — donc impossible à casser hors ligne dans un délai raisonnable.

### Détecter Golden Ticket et Silver Ticket

Un signal typique de Silver Ticket : un logon (Event ID 4624) sans la demande de ticket correspondante (4768/4769) côté contrôleur de domaine — preuve que le ticket n'a jamais transité par le DC comme il le devrait normalement.

En cas de suspicion de Golden Ticket, la remédiation est de réinitialiser le compte `krbtgt` deux fois de suite (avec un délai entre les deux, pour invalider tous les tickets forgés en circulation) :
```powershell
Set-ADAccountPassword -Identity krbtgt -Reset -NewPassword (ConvertTo-SecureString "NouveauMDPComplexe!" -AsPlainText -Force)
```

### Détecter DCSync

Une réplication (`DS-Replication-Get-Changes`) provenant d'un compte qui n'est pas un contrôleur de domaine est anormale par définition :
```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4662} |
  Where-Object { $_.Message -match '1131f6aa-9c07-11d1-f79f-00c04fc2dcd2' }
```

### Cloisonner les privilèges (modèle de tiering)

Séparer les comptes administrateurs par niveau de criticité empêche qu'une compromission d'un poste utilisateur ne remonte jusqu'aux contrôleurs de domaine :
- **Tier 0** : contrôleurs de domaine, comptes Domain Admin
- **Tier 1** : serveurs applicatifs
- **Tier 2** : postes utilisateurs

La règle absolue : un compte Tier 0 ne doit jamais s'authentifier sur une machine Tier 1/2, car son hash y deviendrait récupérable par un attaquant ayant compromis ce poste.

### Événements à surveiller en priorité

| Event ID | Signification |
|---|---|
| 4768 | Demande de TGT (authentification initiale) |
| 4769 | Demande de TGS (accès à un service) |
| 4776 | Validation de credentials (NTLM) |
| 4662 | Accès à un objet AD (réplication, ACL) |
| 5136 | Modification d'un objet AD (ex: ajout à un groupe privilégié) |

## Documentation officielle

- NIST SP 800-63 (Digital Identity Guidelines) : https://pages.nist.gov/800-63-3/
- Microsoft — Event 4769 : https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4769
- gMSA : https://learn.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview
