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
