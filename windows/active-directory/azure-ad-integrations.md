# Azure AD Integrations

## Theory

Azure AD is a popular method to extend identity management from on-premises AD to Microsoft's Azure offerings.

An on-premises AD can be integrated with Azure AD using Azure AD Connect with the following methods:

* Password Hash Sync (PHS)
* Pass-Through Authentication (PTA)
* Federation&#x20;

Azure AD Connect is installed on-premises and has a high privilege account both in on AD and Azure AD!

## Abuse

### PHS

#### Enumerate

Using PowerView

{% code overflow="wrap" %}
```powershell
Get-DomainUser -Identity "<$User with sync> -Domain <$Domain>
```
{% endcode %}

Using AD-Module

We can find out the machine where Azure AD Connect is installed by looking at the Description of special account whose name begins with MSOL\_.

Using AD-Module:

{% code overflow="wrap" %}
```powershell
Get-ADUser -Filter "samAccountName -like 'MSOL_*'" -Server <$Domain> -Properties * | select SamAccountName,Description | fl
```
{% endcode %}

<figure><img src="https://github.com/italianpenty/WriteUps/raw/main/.gitbook/assets/immagine%20(31).png" alt=""><figcaption></figcaption></figure>

#### Get credentials and shell

With admin privileges, if if we run adconnect.ps1, we can extract the credentials of the account in clear text

```powershell
./adconnect.ps1
```

And with the password you can ran a shell as the user using runas

```powershell
runas /user:<$User> /netonly cmd 
```

With that you can [DCSync](exploitation-abuse/dacl/dcsync.md)

{% hint style="info" %}
Note that because AD Connect synchronizes hashes every two minutes, in an Enterprise Environment, the MSOL\_ account will be excluded from tools like MDI! This will allow us to run DCSync without any alerts!
{% endhint %}

## References

{% embed url="https://blog.xpnsec.com/azuread-connect-for-redteam/" %}
