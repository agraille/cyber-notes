---
tags:
  - blue-team
  - defense
---

# Get-WinEvent — Analyse de masse des logs Windows

## C'est quoi

`Get-WinEvent` est la cmdlet PowerShell pour interroger les journaux d'événements Windows en masse — logs locaux, logs distants, ou fichiers `.evtx` exportés. Elle permet de filtrer, parser et analyser des millions d'événements rapidement, bien plus efficace que l'Observateur d'événements manuel.

---

## Lister les logs disponibles

```powershell
# Tous les logs avec propriétés clés
Get-WinEvent -ListLog * | Select-Object LogName, RecordCount, IsClassicLog, IsEnabled, LogMode, LogType | Format-Table -AutoSize
```

Champs importants :
- `LogName` : nom du log
- `RecordCount` : nombre d'événements
- `IsEnabled` : activé ou non
- `LogMode` : Circular, Retain, ou AutoBackup
- `LogType` : Administrative, Operational, Analytical, Debug

## Lister les providers (sources d'événements)

```powershell
# Tous les providers
Get-WinEvent -ListProvider * | Format-Table -AutoSize
```

Montre quels providers alimentent quels logs.

---

## Récupérer des événements — cas simples

### Depuis le journal System (les 50 derniers)

```powershell
Get-WinEvent -LogName 'System' -MaxEvents 50 | Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message | Format-Table -AutoSize
```

### Depuis Sysmon (les 30 derniers)

```powershell
Get-WinEvent -LogName 'Microsoft-Windows-Sysmon/Operational' -MaxEvents 30 | Select-Object TimeCreated, ID, ProviderName, LevelDisplayName, Message | Format-Table -AutoSize
```

### Les événements les plus anciens (au lieu des plus récents)

```powershell
Get-WinEvent -LogName 'Microsoft-Windows-Sysmon/Operational' -Oldest -MaxEvents 30
```

---

## Récupérer depuis un fichier .evtx

```powershell
Get-WinEvent -Path 'C:\path\to\exported.evtx' -MaxEvents 5 | Select-Object TimeCreated, ID, ProviderName, Message | Format-Table -AutoSize
```

Utile pour analyser des logs exportés d'une autre machine ou d'une sauvegarde.

---

## Filtrer avec FilterHashtable

Syntaxe de base :

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    ID = 1,3
} | Select-Object TimeCreated, ID, Message
```

### Filtrer sur plusieurs Event IDs

```powershell
# Event ID 1 (Process Create) et ID 3 (Network Connection)
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; ID=1,3}
```

### Filtrer sur une plage de dates

```powershell
$startDate = (Get-Date -Year 2023 -Month 5 -Day 28).Date
$endDate = (Get-Date -Year 2023 -Month 6 -Day 3).Date

Get-WinEvent -FilterHashtable @{
    LogName='Microsoft-Windows-Sysmon/Operational'
    ID=1,3
    StartTime=$startDate
    EndTime=$endDate
} | Select-Object TimeCreated, ID, Message
```

Note : `EndTime` est **exclusif** — c'est pourquoi on met le 3 juin au lieu du 2.

---

## Parser XML pour extraire des champs spécifiques

Exemple : extraire IP source/destination depuis Sysmon Event ID 3 (Network Connection) :

```powershell
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; ID=3} |
ForEach-Object {
    $xml = [xml]$_.ToXml()
    $eventData = $xml.Event.EventData.Data
    New-Object PSObject -Property @{
        SourceIP = $eventData | Where-Object {$_.Name -eq "SourceIp"} | Select-Object -ExpandProperty '#text'
        DestinationIP = $eventData | Where-Object {$_.Name -eq "DestinationIp"} | Select-Object -ExpandProperty '#text'
        ProcessId = $eventData | Where-Object {$_.Name -eq "ProcessId"} | Select-Object -ExpandProperty '#text'
    }
} | Where-Object {$_.DestinationIP -eq "52.113.194.132"}
```

Le résultat affiche uniquement les connexions vers l'IP suspecte.

---

## Filtrer avec FilterXml

Pour des requêtes plus complexes, utiliser directement du XML :

```powershell
$Query = @"
    <QueryList>
        <Query Id="0">
            <Select Path="Microsoft-Windows-Sysmon/Operational">
                *[System[(EventID=7)]] and *[EventData[Data='mscoree.dll']] or *[EventData[Data='clr.dll']]
            </Select>
        </Query>
    </QueryList>
"@

Get-WinEvent -FilterXml $Query | ForEach-Object {Write-Host $_.Message `n}
```

Utile pour détecter le chargement anormal de `.NET` DLLs (signe d'injection PowerShell non managée).

---

## Filtrer avec FilterXPath

Pour les recherches basées sur des chemins XPath simples :

```powershell
# Chercher Process Create (ID 1) avec ligne de commande contenant "-enc"
Get-WinEvent -LogName 'Microsoft-Windows-Sysmon/Operational' -FilterXPath "*[EventData[Data[@Name='Image']='C:\Windows\System32\reg.exe']]"
```

Détecter l'installation de Sysinternals (acceptation EULA) :

```powershell
Get-WinEvent -LogName 'Microsoft-Windows-Sysmon/Operational' -FilterXPath "*[EventData[Data[@Name='Image']='C:\Windows\System32\reg.exe']] and *[EventData[Data[@Name='CommandLine']='\"C:\Windows\system32\reg.exe\" ADD HKCU\Software\Sysinternals /v EulaAccepted /t REG_DWORD /d 1 /f']]"
```

---

## Filtrer sur des propriétés spécifiques (Where-Object)

Chercher toutes les créations de process où la **ligne de commande parente** contient `-enc` (obfuscation PowerShell) :

```powershell
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; ID=1} | 
Where-Object {$_.Properties[21].Value -like "*-enc*"} | 
Format-List
```

Explication :
- `$_.Properties[21].Value` : accède à la propriété 21 (ParentCommandLine dans Event ID 1)
- `-like "*-enc*"` : cherche n'importe quelle chaîne contenant `-enc`
- `Format-List` : affiche tout sous forme lisible

Autres indices de propriété utiles pour Event ID 1 :
- `[20]` = ParentImage
- `[10]` = CommandLine
- `[9]` = CurrentDirectory

---

## Cas d'usage courants

### Chercher beaconing vers une IP suspecte

```powershell
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; ID=3} |
Where-Object {$_.Message -match "52.113.194.132"} |
Select-Object TimeCreated, @{n='DestIP';e={$_.Message -replace '.*DestinationIp: (\S+).*','$1'}}, Message
```

### Détecter DLL injection (.NET runtime dans processus natif)

```powershell
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-Sysmon/Operational'; ID=7} |
Where-Object {$_.Message -match "clr.dll|clrjit.dll|mscoree.dll"} |
Select-Object TimeCreated, @{n='Image';e={$_.Message -replace '.*Image: (\S+).*','$1'}}, @{n='DLL';e={$_.Message -replace '.*ImageLoaded: (\S+).*','$1'}}
```

### Énumération : tous les Event IDs uniques d'un log

```powershell
Get-WinEvent -LogName 'Microsoft-Windows-Sysmon/Operational' -MaxEvents 10000 | 
Group-Object ID | 
Sort-Object Count -Descending | 
Select-Object Name, Count
```

---

## Points d'attention

- **Performance** : sur gros volumes, utiliser `-MaxEvents` ou des filtres stricts (dates, ID spécifique)
- **Propriétés XML** : l'index des propriétés change selon l'Event ID — vérifier dans l'Observateur d'événements ou via `Select-Object -Property *`
- **Filtrage XPath vs Hashtable** : hashtable est plus lisible pour filtres simples, XPath/XML pour requêtes complexes
- **Fichiers .evtx** : toujours spécifier `-Path`, pas `-LogName`, pour les fichiers exportés
- **Exécution distante** : `Get-WinEvent` peut interroger des logs distants avec `-ComputerName`

---

## Documentation

- Microsoft Get-WinEvent : https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent
- Sysmon Event IDs : https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon
- Windows Event Log XML : https://learn.microsoft.com/en-us/windows/win32/wes/eventschema
alité « D
