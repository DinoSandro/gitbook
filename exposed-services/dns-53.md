# DNS 53

### **Enumeration**

```bash
dig axfr @<IP> <Domain Name>
```

<figure><img src="../.gitbook/assets/Pasted image 20230508172556.png" alt=""><figcaption></figcaption></figure>

### **DNS HIJACKING**

If through LFI you manage to dump the secret key of the DNS present in

```PATH
/etc/bind/named.conf
```

<figure><img src="../.gitbook/assets/Pasted image 20230508173833.png" alt=""><figcaption></figcaption></figure>

You can try substituting a subdomain to redirect your traffic to you

Export the key in the format

```BASH
export HMAC=<Algorithm>:<Key Name>:<Key>
```

Use the command

```bash
nsupdate -y $HMAC
```

to modify the dns

## References

&#x20;[DNS Updates with nsupdate - Serverless Industries](https://serverless.industries/2020/09/27/dns-nsupdate-howto.en.html)
