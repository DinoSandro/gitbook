# AirCrack-ng

### Airmon-ng

Airmon-ng is a convenient way to enable and disable monitor mode on various wireless interfaces.

Running airmon-ng without any parameters displays the status and information about the wireless interfaces on the system.

It will be important to identify and terminate processes like Network Manager. The **Airmon-ng check** parameter checks for, and lists, these processes.

```
sudo airmon-ng check
```

Airmon-ng with the check kill parameter first tries to stop known services gracefully then kills the rest of the processes:

```
sudo airmon-ng check kill
```

To place our wlan0 wireless interface in monitor mode, let's execute Airmon-ng with the start option and the interface name.

```
sudo airmon-ng start wlan0
```

### Airodump-ng

The options we will use most often are saving to a file, filtering by BSSID, and capturing only on a specific channel.

| Option        | Description                                                 |
| ------------- | ----------------------------------------------------------- |
| -w prefix     | Saves the capture dump to the specified filename            |
| --bssid BSSID | Filters Airodump-ng to only capture the specified BSSID     |
| -c channel(s) | Forces Airodump-ng to only capture the specified channel(s) |

#### Sniffing with Airodump-ng

With our wireless interface in monitor mode, we initiate our first sniffing session with airodump-ng, the interface name, and the -c option to only capture traffic on channel 2.

```
sudo airodump-ng wlan0mon -c 2
```

#### **Airodump-ng Fields**

| Field   | Description                                                                                                                                                                                                                      |
| ------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| BSSID   | The MAC address of the AP                                                                                                                                                                                                        |
| PWR     | The signal level reported by the card, which will get higher as we get closer to the AP or station                                                                                                                               |
| RXQ     | Receive Quality as measured by the percentage of frames successfully received over the last 10 seconds                                                                                                                           |
| Beacons | Number of announcement frames sent by the AP                                                                                                                                                                                     |
| # Data  | Number of captured data packets (if WEP, this is the unique IV count), including data broadcast packets                                                                                                                          |
| #/s     | Number of data packets per second measured over the last 10 seconds                                                                                                                                                              |
| CH      | Channel number taken from beacon frames. Note that sometimes frames from other channels are captured due to overlapping channels                                                                                                 |
| MB      | Maximum speed supported by the AP. 11=802.11b, 22=802.11b+, up to 54 is 802.11g and anything higher is 802.11n or 802.11ac                                                                                                       |
| ENC     | Encryption algorithm in use. OPN=no encryption, "WEP?"=WEP or higher (not enough data to choose between WEP and WPA/WPA2), WEP=static or dynamic WEP, and WPA or WPA2 if TKIP or CCMP is present. WPA3 and OWE both require CCMP |
| CIPHER  | The cipher detected: CCMP, WRAP, TKIP, WEP, WEP40, or WEP104                                                                                                                                                                     |
| AUTH    | The authentication protocol used. One of MGT (WPA/WPA2/WPA3 Enterprise), SKA (WEP shared key), PSK (WPA/WPA2/WPA3 pre shared key), or OPN (WEP open authentication)                                                              |
| ESSID   | The so-called SSID, which can be empty if the SSID is hidden                                                                                                                                                                     |

The table below contains descriptions of all the Airodump-ng fields in the bottom section, where all the stations, associated and unassociated, are displayed.

| Field   | Description                                                                      |
| ------- | -------------------------------------------------------------------------------- |
| BSSID   | The MAC address of the AP                                                        |
| STATION | The MAC address of each associated station                                       |
| Rate    | Station's receive rate, followed by transmit rate                                |
| Lost    | Number of data frames lost over the last 10 seconds based on the sequence number |
| Packets | Number of data packets sent by the client                                        |
| Probes  | The ESSIDs probed by the client                                                  |

#### Airodump-ng Troubleshooting

Let's quickly review a few ways to solve common issues that we might encounter while using Airodump-ng.

**No APs or Clients are Shown**

When no APs or clients are shown in the Airodump-ng output, we should first verify that there are APs on the current channel. Next, we can make sure our card works in managed mode. If we still can't see any APs or clients, we can try unloading the driver with rmmod and reloading it with modprobe. It could be the case that we need to unload multiple modules. Finally, we can check dmesg for errors.

**Little or No Data Being Captured**

If little or no data is being captured during our sniffing session, we can start by making sure that we used the -c or --channel option to specify a single channel. If we don't do this, Airodump-ng will hop between different channels. We might also consider our physical location. We may need to be physically closer to (or farther away from) the AP to get a quality signal. Next, we can check to make sure that we started our wireless card in monitor mode with Airmon-ng. Finally, we can ensure there is no network manager causing interference. We can do this by running airmon-ng check.

**Airodump-ng Stops Capturing After a Short Period of Time**

If Airodump-ng stops capturing after a short period of time, the most common cause is that a connection manager is running on the system that takes the wireless card out of monitor mode.

To remedy this, we'll need to use airmon-ng check kill prior to placing our card in monitor mode. We can also make sure that no interfering processes are running on your system by using airmon-ng check.

Another possibility is firmware issues. The firmware, being a piece of compiled code, may crash, which would interrupt the capture process. Such issues will typically be logged in dmesg.

**SSIDs Displayed as "\<length: ?>"**

At times, we might see "\<length: ?>" as the SSID on the Airodump-ng display. The "?" is normally the length of the hidden SSID. For example, if the SSID was test123 then it would show up as "\<length: 7>" where 7 is the number of characters. When the length is 0 or 1, the AP will not reveal the actual length. A "?" indicates a wireless frame contained a new BSSID. In these cases, it may be the case that a wireless device is communicating with an AP too far away for us to hear.

**"Fixed channel" Error Message**

If we, for example, set the channel to channel 6 with the -c option, we might encounter an error message similar to the one found here.

```
 CH  6 ][ Elapsed: 28 s ][ 2015-10-21 16:29 ][ fixed channel wlan0mon: 1
```

> Listing 15 - Fixed channel message

When we receive this error, we know that some other process is changing to channel 1 or that some process is channel scanning.

We need to eliminate the root cause and restart Airodump-ng. The problem is most likely the result of network managers and other interfering processes that are running. If this is the case, we can resolve the issue by killing the network managers with airmon-ng check kill.

This error message might also mean that we cannot use this channel (and Airodump-ng failed to set the channel). For example, if we tried setting the channel to channel 40 with a card that only supports channels 1 to 11.

**No Output Files**

We ran Airodump-ng and now cannot find the output files.

First, we'll want to make sure we ran airodump-ng with the option to create output files with -w or --write and the filename prefix. Without this option, Airodump-ng won't create output files.

By default, the output files are placed in the directory where Airodump-ng is run. Before starting Airodump-ng, we can use pwd to display the current directory. We'll make a note of this directory so we can return to it later.

### Aireplay-ng

Aireplay-ng supports the following attacks. They are listed along with the corresponding number from the tool's documentation.

| Attack # | Attack Name                          |
| -------- | ------------------------------------ |
| 0        | Deauthentication                     |
| 1        | Fake Authentication                  |
| 2        | Interactive Packet Replay            |
| 3        | ARP Request Replay Attack            |
| 4        | KoreK ChopChop Attack                |
| 5        | Fragmentation Attack                 |
| 6        | Café-Latte Attack                    |
| 7        | Client-Oriented Fragmentation Attack |
| 8        | WPA Migration Mode Attack            |
| 9        | Injection Test                       |

The majority of these options are attacks specific to WEP networks.&#x20;

#### Aireplay-ng Injection Test

Prior to running a basic injection test, it is important that we first set our card to the desired channel. For this test, our target is on channel 3.

```
sudo airmon-ng start wlan0 3
```

```
sudo aireplay-ng -9 wlan0mon
```

**Card-to-Card (Attack) Injection Test**

```
sudo aireplay-ng -9 -i wlan1mon wlan0mon
```

<figure><img src="../.gitbook/assets/immagine (2).png" alt=""><figcaption></figcaption></figure>

### Airdecap-ng

Airdecap-ng is useful after we have successfully retrieved the key to a wireless network. We can use it to decrypt WEP, WPA PSK, or WPA2 PSK capture files. We'll use it to strip wireless headers from an unencrypted wireless capture.

### Airgraph-ng

Airgraph-ng is a Python script that can be used to create graphs of wireless networks using the CSV files generated by Airodump-ng.
