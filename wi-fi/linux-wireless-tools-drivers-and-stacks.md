# Linux Wireless Tools, Drivers, and Stacks

### Loading and Unloading Wireless Drivers

**airmon-ng** command in the command prompt to determine our device's driver. Airmon-ng is a utility from the Aircrack-ng suite of tools for auditing Wi-Fi networks.



Next, let's run the **lsusb** command. This command lists a system's USB devices and shows detailed information for each device:



**lsmod** lists all the loaded modules as well as the dependencies of each module.



### Wireless Tools

There are two sets of tools to set, show, or change wireless card parameters. _iw_, the modern set of tools, are made for the newer mac80211 framework. On the other side, _iwconfig_ (and others, such as _iwpriv_), dating back from the early 2000's, were made for the ieee80211 framework. While iwconfig can still be used for some of the mac80211 features, they are deprecated and limited compared to the capabilities of iw.

#### iwconfig and Other Utilities

* _iwconfig_ manipulates the basic wireless parameters: change modes, set channels, and keys.
* _iwlist_ allows for the initiation of scanning, listing frequencies, bit rates, and encryption keys.
* _iwspy_ provides per-node link quality (not often implemented by drivers).
* _iwpriv_ allows for the manipulation of the Wireless Extensions specific to a driver.

#### The iw Utility

Assuming the drivers have been loaded properly, running iw list will provide us with lots of detailed information about the wireless devices and their capabilities

To get a listing of wireless access points that are within range of our wireless card, we will use iw with the dev wlan0 option, which specifies our device. Next, we'll add the scan parameter. We then pipe this command through grep SSID to filter our output to only wireless network names:

<figure><img src="../.gitbook/assets/immagine.png" alt=""><figcaption></figcaption></figure>

