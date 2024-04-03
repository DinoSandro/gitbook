# SNMP 161

### **ENUMERATION**

#### Community String

Find the community string used for authentication.

{% code overflow="wrap" %}
```bash
onesixtyone <MACHINE-IP> -c /usr/share/seclists/Discovery/SNMP/snmp-onesixtyone.txt
```
{% endcode %}

<figure><img src="../.gitbook/assets/Pasted image 20230503160242.png" alt=""><figcaption></figcaption></figure>

#### **DUMP**

Dump all the informations

```bash
snmp-check <IP> -c <Community String>
```

<figure><img src="../.gitbook/assets/Pasted image 20230503161047.png" alt=""><figcaption></figcaption></figure>
