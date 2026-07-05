# Hardening Windows

## 1. Le concept

### Où ça se situe dans NIST

Le durcissement système relève de la fonction **Protect (PR)** du NIST CSF, principalement des catégories **PR.AC (Access Control)** et **PR.PT (Protective Technology)**. Le principe sous-jacent que pousse le NIST est celui du **moindre privilège et de la réduction de surface d'attaque** : chaque fonctionnalité, protocole ou privilège actif par défaut qui n'est pas strictement nécessaire est une opportunité offerte à un attaquant.

### C'est quoi concrètement

Le durcissement Windows consiste à désactiver les protocoles hérités exploitables, limiter les privilèges disponibles, et appliquer des configurations restrictives à l'échelle du parc via GPO (Group Policy Objects), plutôt que de compter uniquement sur des outils de détection pour réagir après coup.

## 2. Pourquoi ça marche (le mécanisme)

La majorité des attaques qui touchent un environnement Windows/Active Directory n'exploitent pas une faille logicielle complexe, mais des **défauts de configuration par défaut**, conçus à une époque où la compatibilité ascendante primait sur la sécurité :

- **SMBv1** contient des failles connues et exploitables (comme celle utilisée par WannaCry), mais reste souvent activé par défaut pour la compatibilité avec de très vieux systèmes
- **NTLM** est un protocole d'authentification plus faible que Kerberos, vulnérable au relais (un attaquant intercepte une tentative d'authentification et la rejoue vers un autre système)
- **LLMNR/NBT-NS** sont des protocoles de résolution de noms qui répondent à n'importe quelle requête sur le réseau local, ce qu'un attaquant peut détourner pour se faire passer pour un serveur légitime et intercepter des identifiants
- La **réutilisation du même mot de passe administrateur local** sur tout le parc signifie qu'une seule machine compromise donne accès à toutes les autres (technique dite du "pass-the-hash")

Désactiver ce qui n'est pas nécessaire et cloisonner les privilèges élimine ces chemins d'attaque à la racine, plutôt que d'espérer les détecter une fois exploités.

## 3. Mise en œuvre — le chemin concret

### Désactiver les protocoles legacy

**SMBv1** :
```powershell
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol
```

**NTLM** — forcer Kerberos et restreindre NTLM au strict nécessaire (GPO) :
```
Computer Configuration > Windows Settings > Security Settings >
Local Policies > Security Options >
"Network security: Restrict NTLM: NTLM authentication in this domain" = Deny all
```

**LLMNR/NBT-NS** :
```powershell
# Désactiver LLMNR
Set-ItemProperty -Path "HKLM:\Software\Policies\Microsoft\Windows NT\DNSClient" -Name "EnableMulticast" -Value 0

# Désactiver NetBIOS sur toutes les interfaces
Get-WmiObject Win32_NetworkAdapterConfiguration | ForEach-Object { $_.SetTcpipNetbios(2) }
```

### Éliminer la réutilisation de mots de passe locaux avec LAPS

LAPS génère un mot de passe administrateur local unique et régulièrement changé sur chaque machine, cassant la chaîne de propagation d'un pass-the-hash :

```powershell
Install-WindowsFeature -Name RSAT-AD-PowerShell
Update-LapsADSchema
Set-LapsADComputerSelfPermission -Identity "OU=Workstations,DC=corp,DC=local"

# Consultation d'un mot de passe LAPS
Get-LapsADPassword -Identity PC01 -AsPlainText
```

### Autres contrôles à auditer

Renforcer la journalisation PowerShell, souvent utilisé comme vecteur d'attaque en "living off the land" :
```powershell
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" -Name "EnableScriptBlockLogging" -Value 1
```

Activer les règles de réduction de surface d'attaque (ASR) de Windows Defender, par exemple pour bloquer les applications Office qui créent des processus enfants (technique classique de macro malveillante) :
```powershell
Set-MpPreference -AttackSurfaceReductionRules_Ids D4F940AB-401B-4EfC-AADC-AD5F3C50688A -AttackSurfaceReductionRules_Actions Enabled
```

## Documentation officielle

- NIST Cybersecurity Framework : https://www.nist.gov/cyberframework
- Windows LAPS : https://learn.microsoft.com/en-us/windows-server/identity/laps/laps-overview
- Attack Surface Reduction rules : https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/attack-surface-reduction-rules-reference
- CIS Benchmarks (référence de durcissement détaillée) : https://www.cisecurity.org/cis-benchmarks
