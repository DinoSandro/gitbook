# Privescs

### Look for password int he registry

```powershell
reg query HKLM /f password /t REG_SZ /s
```

```powershell
cmdkey /list
```

### Automatics

_Tools_

* [enjoiz/Privesc: Windows batch script that finds misconfiguration issues which can lead to privilege escalation. (github.com)](https://github.com/enjoiz/Privesc)
* [PowerShellMafia/PowerSploit: PowerUP. (github.com)](https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc)
* [AlessandroZ/BeRoot: Privilege Escalation Project - Windows / Linux / Mac (github.com)](https://github.com/AlessandroZ/BeRoot)
* [carlospolop/PEASS-ng: PEASS - Privilege Escalation Awesome Scripts SUITE (with colors) (github.com)](https://github.com/carlospolop/PEASS-ng)

#### **FULL ENUMERATION**

```Powerup
Invoke-AllChecks
```

```Privesc
Invoke-PrivEsc
```

#### **SERVICES**

_Unquoted paths and space in the name_

```Powerup
Get-ServiceUnquoted -Verbose
```

_Write in binary path_

```powerup
Get-ModifiableServiceFile -Verbose
```

_configuration current user can modify_

```powerup
Get-ModifiableService -Verbose
```
