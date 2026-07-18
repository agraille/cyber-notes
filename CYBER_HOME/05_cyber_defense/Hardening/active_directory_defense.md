# Active Directory Defense

## C'est quoi

**Active Directory** est le système d'identité et de contrôle d'accès central en environnement Windows/entreprise. C'est la cible numéro 1 en pentest : compromettre l'AD = accès à toute l'infrastructure. La défense repose sur trois piliers :

1. **Détecter les attaques Kerberos** (Kerberoasting, Golden Ticket, DCSync)
2. **Limiter la propagation** via modèle de tiering (Tier 0/1/2)
3. **Surveiller les Event IDs critiques** qui trahissent un abus

**Attaques courantes** :
- **Kerberoasting** (T1558.003) : dump TGS des comptes de service, crack offline
- **Golden Ticket** (T1558.001) : forge un TGT valide en utilisant le hash krbtgt
- **Silver Ticket** (T1558.002) : forge un TGS sans passer par le DC
- **DCSync** (T1033) : réplication non-autorisée des hashes du domaine

---

## Mise en œuvre — le chemin concret

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
