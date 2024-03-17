# Privileges

### SeTakeOwnershipPrivilege

Search for a process that is of the same architecture as lsass and has the same authority.

```powershell
ps
```

Look for the pid transfer your shell in this process

```powershell
migrate $PID
```

