# Hardening Windows

## C'est quoi

Le durcissement Windows consiste à **désactiver les vieux protocoles exploitables** par défaut, limiter les privilèges, et appliquer des configurations restrictives via GPO. L'objectif : réduire la surface d'attaque plutôt que de compter sur la détection après.

**Failles par défaut courantes** :
- **SMBv1** (WannaCry, EternalBlue) — contient des RCE, reste souvent activé pour compatibilité
- **NTLM** (pass-the-hash, relay attacks) — plus faible que Kerberos
- **LLMNR/NBT-NS** (responder, poisoning) — répond à n'importe quelle requête réseau
- **Même mot de passe local partout** (pass-the-hash dans le parc) — une machine = toutes les machines

---

## Mise en œuvre — le chemin concret

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
