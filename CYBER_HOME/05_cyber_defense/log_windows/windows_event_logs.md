---
tags:
  - blue-team
  - defense
---

# Windows Event Logs & Finding Evil

## Journaux disponibles

Accessibles via l'Observateur d'événements (`eventvwr.msc`), sous "Windows Logs" :

- **Application** : erreurs/infos des logiciels installés
- **Security** : authentifications, accès objets, changements d'audit — le plus important en investigation
- **Setup** : activités d'installation/configuration
- **System** : activité de l'OS
- **Forwarded Events** : logs centralisés depuis d'autres machines

Les fichiers `.evtx` déjà enregistrés se chargent dans "Saved Logs".

## Anatomie d'un événement

| Champ                           | Contenu                                  |
|---------------------------------|------------------------------------------|
| Nom du journal                  | Application, Security, System...         |
| Source                          | Composant qui a généré l'événement       |
| Event ID                        | Identifiant du type d'événement          |
| Task Category                   | Contexte fonctionnel                     |
| Level                           | Information, Warning, Error, Critical, Verbose |
| Keywords                        | Classification large (Audit Success/Failure) |
| User                            | Compte connecté au moment de l'événement  |
| Logon ID                        | Identifiant de session — permet de relier tous les événements d'une même session |
| Computer                        | Machine source                            |

## Activer une journalisation détaillée

```powershell
# Ligne de commande dans les créations de process (Event ID 4688)
auditpol /set /subcategory:"Process Creation" /success:enable
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\Audit" /v ProcessCreationIncludeCmdLine_Enabled /t REG_DWORD /d 1
```

## Requête XML personnalisée

Filtrer le journal actuel > XML > Modifier la requête manuellement.

Pivoter sur un Logon ID pour reconstruire toute l'activité d'une session :

```xml
<QueryList>
  <Query Id="0" Path="Security">
    <Select Path="Security">
      *[EventData[Data[@Name='SubjectLogonId']='0x3E7']]
    </Select>
  </Query>
</QueryList>
```

Filtrer sur un Event ID spécifique en plus d'un champ custom :

```xml
<QueryList>
  <Query Id="0" Path="Security">
    <Select Path="Security">
      *[System[(EventID=4663)]] and *[EventData[Data[@Name='ObjectName']='C:\Windows\System32\config\SAM']]
    </Select>
  </Query>
</QueryList>
```

Event avec fenetre de temps
```xml
<QueryList>
  <Query Id="0" Path="Security">
    <Select Path="Security">
      *[System[(EventID=4907) and TimeCreated[@SystemTime&gt;='2022-08-03T17:20:00.000Z' and @SystemTime&lt;='2022-08-03T17:26:00.000Z']]]
    </Select>
  </Query>
</QueryList>
```

## Récupérer les logs en ligne de commande

```powershell
# Requête par ID sur le journal Security
Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4624}

# Requête par plage horaire
Get-WinEvent -FilterHashtable @{LogName='Security'; StartTime=(Get-Date).AddHours(-1)}

# Export d'un evtx pour analyse externe
wevtutil epl Security C:\export\security.evtx

# Lecture d'un evtx exporté
Get-WinEvent -Path C:\export\security.evtx
```

Avec `wevtutil` (utile en post-exploitation ou hors GUI) :

```bash
wevtutil qe Security /q:"*[System[(EventID=4624)]]" /f:text /c:20
```

## Table des Event IDs — System

| Event ID | Signification |
|---|---|
| 1074 | Arrêt/redémarrage du système |
| 6005 | Démarrage du service Journal d'événements |
| 6006 | Arrêt du service Journal d'événements |
| 6013 | Temps de fonctionnement du système (uptime) |
| 7040 | Changement de type de démarrage d'un service |

## Table des Event IDs — Security

| Event ID | Signification |
|---|---|
| 1102 | Journal d'audit effacé (couverture de traces) |
| 1116 | Détection de malware par l'antivirus |
| 1118 | Début de remédiation antivirus |
| 1119 | Remédiation antivirus réussie |
| 1120 | Remédiation antivirus échouée |
| 4624 | Logon réussi |
| 4625 | Logon échoué |
| 4648 | Logon avec identifiants explicites (mouvement latéral) |
| 4656 | Demande de handle sur un objet |
| 4662 | Accès à un objet AD |
| 4663 | Tentative d'accès à un objet |
| 4672 | Logon avec privilèges spéciaux |
| 4688 | Création de process |
| 4698 | Création d'une tâche planifiée |
| 4700 / 4701 | Activation / désactivation d'une tâche planifiée |
| 4702 | Mise à jour d'une tâche planifiée |
| 4719 | Modification de la politique d'audit système |
| 4720 | Création d'un compte utilisateur |
| 4738 | Modification d'un compte utilisateur |
| 4768 | Demande de TGT Kerberos |
| 4769 | Demande de TGS Kerberos |
| 4771 | Échec de pré-authentification Kerberos |
| 4776 | Validation d'identifiants par le DC |
| 4907 | Modification de SACL sur un objet |
| 5001 | Modification de la protection temps réel antivirus |
| 5140 | Accès à un partage réseau |
| 5142 | Création d'un partage réseau |
| 5145 | Vérification d'accès à un partage réseau |
| 5157 | Connexion bloquée par la Plateforme de filtrage Windows |
| 7045 | Installation d'un nouveau service |

## Types de Logon (LogonType)

| Valeur | Type |
|---|---|
| 2 | Interactive (console) |
| 3 | Network (ex: accès SMB) |
| 4 | Batch |
| 5 | Service |
| 7 | Unlock |
| 8 | NetworkCleartext |
| 9 | NewCredentials (RunAs) |
| 10 | RemoteInteractive (RDP) |
| 11 | CachedInteractive |

## Méthode de pivot rapide

1. Repérer un event ID suspect (ex: 4907, 1102, 7045)
2. Récupérer son `Logon ID` et son horodatage
3. Filtrer via requête XML sur ce Logon ID pour isoler la session complète
4. Ordonner chronologiquement : logon → actions → logoff
5. Identifier le `ProcessName`/`ProcessID` responsable de chaque action

## Documentation

- Référence complète des événements de sécurité Microsoft : https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/
- Filtrage XML avancé : https://learn.microsoft.com/en-us/windows/win32/wes/consuming-events
