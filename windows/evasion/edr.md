# EDR

Endpoint Detection and Response (EDRs) systems protect individual devices by continuosly monitoring for and responding to security threats.

{% hint style="info" %}
To avoid detection you need to gel in: act normal to look normal
{% endhint %}

## Microsoft Defender for Endpoint (MDE)

### Credential extraction

#### Theory

Simply dump the credential from LSASS will be detected\
Most tools create an LSASS dump by:

1. Gaining a handle to the LSASS process.
2. Creating a minidump using the MiniDumpWriteDump WinAPI function implemented in dbghelp.dll / dbgcore.dll.
3. Writing the dump file on disk

Do dump the LSASS stealthy a different process injection method need to be used like MockingJay

In short, this technique involves invoking our shellcode within a memory area that naturally allows Read-Write-Execute (RWX) permissions in a trusted DLL without having to use typical process injection Windows/NT APIs like VirtualAllocEx (heavily monitored by AV/EDRs). \
Two possibilities of performing injection exist:&#x20;

* Self Injection: Find a vulnerable trusted DLL with RWX permissions to copy our shellcode into and load the DLL and execute shellcode.
* Remote Injection: Leveraging trusted applications that use a vulnerable trusted DLL with RWX permissions to perform the same functionality as in Self Injection.

To find a vulnerabile DDL you can use [RWXfinder](https://github.com/pwnsauc3/RWXfinder)

Using the MockingJay technique with [NanoDump ](https://github.com/fortra/nanodump)is the best idea to stay cover

{% hint style="danger" %}
If you save the dump on the target in a valid format it will be flagged. The trick is to save it in a invalid format and restore it later
{% endhint %}

#### Abuse



### File Tranfer

#### Theory

A file transfer using HTTP(S) can be risky and increase the chance of detection.

#### Abuse

To avoid detecion various technique can be user:

* Use a browser to download file instead of command line
* Use SMB, Copy or others file transfer protocol

### Breaking Detection Chain

Most of the EDRs correlate activity in a specific time interval after which it is reset.\
Wait like 10 minutes between queries.



## References

{% embed url="https://www.securityjoes.com/post/process-mockingjay-echoing-rwx-in-userland-to-achieve-code-execution" %}
