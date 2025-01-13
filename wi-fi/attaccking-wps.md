# Attaccking WPS

WPS PINs are typically eight digits long. The last digit is a checksum, which leaves ten million possible PINs. Checking a PIN usually takes between one and three seconds. Brute forcing would take three to four months at best, making it almost pointless; however, we can alter our brute force attack and make it a little more efficient because of how the PIN is verified.

### WPS Attack

After putting our wlan0 interface in monitor mode, we now have an interface called _wlan0mon_.

We will use a tool called _wash_ to give us information about the AP, and use the wlan0mon interface with -i

```
wash -i wlan0mon
```

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

The first column is the BSSID, and the second one is the channel, followed by the signal level reported by the card. The _WPS_ column represents the WPS version. Version 2 mandated mitigations to prevent brute forcing, which, depending on the implementation, may just slow down a bruteforce attack. The _Lck_ indicates if WPS is locked, meaning an attack is pointless at this time. The _Vendor_ column indicates the wireless chipset vendor, which is sometimes advertised in the beacon. The last column is the ESSID of the AP.

We will now use reaver to attack our _wifu_ AP. We have to specify the BSSID of the AP we gathered earlier using wash with -b, the wireless interface using -i, and a very verbose output with -vv.

```
sudo reaver -b 34:08:04:09:3D:38 -i wlan0mon -v
```

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

The method described here will take a long time, and even longer in the event the AP has countermeasures. Earlier, we described the PixieWPS attack. When successful, it will give us results much more quickly. To try it, we will add an additional option to the reaver command, and use -K.

```
sudo reaver -b 34:08:04:09:3D:38 -i wlan0mon -v -K
```



