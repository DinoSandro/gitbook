# Enumeration

### Look inside the bin

```powershell
cd 'C:\$Recycle.bin\<SID>'
```

### Priviledges

```powershell
whoami /priv
```

### Misconfiguration

{% tabs %}
{% tab title="PowerUp.ps1" %}
```powershell
Invoke-AllCheck
```
{% endtab %}

{% tab title="Privesc.ps1" %}
```powershell
Invoke-PrivEsc
```
{% endtab %}
{% endtabs %}

### Services Misconfiguration

{% hint style="info" %}
All done with PowerUp.ps1
{% endhint %}

#### _Unquoted paths and space in the name_

```powershell
Get-ServiceUnquoted -Verbose
```

#### _Write in binary path_

```powershell
Get-ModifiableServiceFile -Verbose
```

#### _Configuration current user can modify_

```powershell
Get-ModifiableService -Verbose
```

## References

[enjoiz/Privesc: Windows batch script that finds misconfiguration issues which can lead to privilege escalation. (github.com)](https://github.com/enjoiz/Privesc)

[PowerShellMafia/PowerSploit: PowerUP. (github.com)](https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc)
