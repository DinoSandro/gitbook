---
description: Mostly done with powerview
---

# Domain Information

### _Get current domain_

```PowerView
Get-Domain
```

<figure><img src="../../../.gitbook/assets/Pasted image 20230606170600.png" alt="" width="554"><figcaption></figcaption></figure>

### _Get object of another domain_

```PowerView
Get-Domain -Domain moneycorp.local
```

<figure><img src="../../../.gitbook/assets/Pasted image 20230606172106.png" alt="" width="507"><figcaption></figcaption></figure>

### _Get domain SID for the current domain_

```PowerView
Get-DomainSID
```

### Get my SID

```powershell
whoami /all | Select-String -Pattern "$USER" -Context 2,0
```

### _Get domain policy for the current domain_

```PowerView
Get-DomainPolicyData
```

<figure><img src="../../../.gitbook/assets/Pasted image 20230606172339.png" alt=""><figcaption></figcaption></figure>

### _Get domain controllers for the current domain_

```PowerView
Get-DomainController
```

<figure><img src="../../../.gitbook/assets/Pasted image 20230606173931.png" alt=""><figcaption></figcaption></figure>

### _Get user information_

```PowerView
Get-DomainUser
```

### _Get domain computers information_

```PowerView
Get-DomainComputer
```

### _Get domain groups information_

```PowerView
Get-NetGroup
```

### _Find shares on hosts in current domain_

```PowerView
Invoke-ShareFinder –Verbose
```

### _Find sensitive files on computers in the domain_

```PowerView
Invoke-FileFinder –Verbose
```

### _Get all fileservers of the domain_

```PowerVieW
Get-NetFileServer
```

#### **GPO ENUMERATION**

```PowerView
Get-DomainGPO
```

```AD_module
GET-NetGPO
```

#### **ACL ENUMERATION**

```PowerView
Get-DomainObjectAcl
```

```powerview
Find-InterestingDomainAcl -ResolveGUIDs |  ?{$_.IdentityReferenceName -match "<USER>"}
```

```powerview
Find-InterestingDomainAcl -ResolveGUIDs |  ?{$_.IdentityReferenceName -match "<GROUP>"}
```

```PowerView
Find-InterestingDomainAcl -ResolveGUIDs
```

### **Trust Enumeration**

```PowerView
Get-DomainTrust
```

#### Identify external trusts in the domain

```Powerview
 Get-DomainTrust | ?{$_.TrustAttributes -eq "FILTER_SIDS"}
```

### **Forest Enumeration**

```PowerView
Get-Forest
```

```AD_module
Get-ForestTrust
```

list only the external trusts int the current forest

{% code overflow="wrap" %}
```powershell
Get-ForestDomain -Verbose | Get-DomainTrust | ?{$_.TrustAttributes -eq 'FILTER_SIDS'}
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/immagine (6).png" alt=""><figcaption></figcaption></figure>

IF trust is bidirectional you can enumerate the trust of the other domain

```
Get-ForestTrust -Forest <$Target Forest>
```

[![](https://github.com/italianpenty/WriteUps/raw/main/.gitbook/assets/immagine%20\(1\)%20\(1\).png)](https://github.com/italianpenty/WriteUps/blob/main/.gitbook/assets/immagine%20\(1\)%20\(1\).png)

### **Search SPN Accounts**

```powerview
Get-DomainUser -SPN
```

#### Over another domain

with AD-Module

{% code overflow="wrap" %}
```powershell
Get-ADTrust -Filter 'IntraForest -ne $true' | %{Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName -Server $_.Name}
```
{% endcode %}

### **Delegations**

#### _Unconstrained_

```powerview
Get-DomainComputer -Unconstrained | select -ExpandProperty name
```

With AD-Module

```
Get-ADComputer -Filter {TrustedForDelegation -eq $True}
```

#### _Constrained_

```powerview
Get-DomainUser -TrustedToAuth
```

to another domain with AD-Module:

{% code overflow="wrap" %}
```powershell
Get-ADObject -Filter {msDS-AllowedToDelegateTo -ne "$null"} -Properties msDS-AllowedToDelegateTo -Server <$Target>
```
{% endcode %}

### Domain users

```powershell
Get-NetUser | select cn
```

## References

{% embed url="https://gist.githubusercontent.com/HarmJ0y/184f9822b195c52dd50c379ed3117993/raw/e5e30c942adb2347917563ef0dafa7054882535a/PowerView-3.0-tricks.ps1" %}
PowerView CheatSheet
{% endembed %}

