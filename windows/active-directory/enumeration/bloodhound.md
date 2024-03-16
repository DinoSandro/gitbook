# BloodHound

## Collectors

You need to use the his collectors to enumerate all the information

Use SharpHound.exe or SharpHound.ps1

<pre class="language-powershell"><code class="lang-powershell"><strong>./SharpHound.exe --collectionmethods All --outputdirectory C:\Users\Public --zipfilename loot.zip
</strong></code></pre>

<figure><img src="../../../.gitbook/assets/Pasted image 20240315100026.png" alt=""><figcaption></figcaption></figure>

To avoid detection

```powershell
Invoke-Bloodhound -CollectionMethod All -ExcludeDC
```

## References

{% embed url="https://support.bloodhoundenterprise.io/hc/en-us/sections/17274879117083-BloodHound-Enterprise-Collection" %}

