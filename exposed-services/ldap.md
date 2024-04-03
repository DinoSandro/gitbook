# LDAP

### **LDAPSEARCH**

Get a Kerberos ticket, modify /etc/krb5.conf, and use the command

```BASH
kinit <Username>
```

to check

```BASH
klist
```

<figure><img src="../.gitbook/assets/Pasted image 20230621113454.png" alt=""><figcaption></figcaption></figure>

Now you can use ldapsearch

```BASH
ldapsearch -H ldap://<MACCHINA< -D '<DOMINIO>\<USER>' -w '<USER>' -Y GSSAPI - b "cn=users,dc=scrm,dc=local" | grep -i "objectSid::" | cut -d ":" -f3
```

### **DOMAIN DUMP**

{% hint style="info" %}
With access as system
{% endhint %}

```BASH
ldapdomaindump $IP -u '<DOMAIN NAME>\<USER>' -p <PASS> --no-json --no-grep
```

<figure><img src="../.gitbook/assets/Pasted image 20230510114519.png" alt=""><figcaption></figcaption></figure>

#### Silver Ticket

With the silver ticket you can dump ldap

```bash
crackmapexec ldap <Domain Name> -u <USER> -k --kdcHost <DC> --users <IP>
```
