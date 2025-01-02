# Wireshark

Wireshark will capture Ethernet packets by default, even when we are using a wireless interface. In order to collect only raw wireless frames, the Wi-Fi adapter must be put in monitor mode prior to launching Wireshark.

```
sudo ip link set wlan0 down
sudo iwconfig wlan0 mode monitor
sudo ip link set wlan0 up
```

Wireshark doesn't _channel hop_ It will stay on whatever channel the wireless adapter is currently on. To quickly scan all channels on 2.4GHz, we can run the following shell script in the background in a terminal.

```
for channel in 1 6 11 2 7 10 3 8 4 9 5
do
  iw dev wlan0mon set channel ${channel}
  sleep 1
done
```

We could also use _airodump-ng_[2](https://portal.offsec.com/courses/pen-210-9545/learning/wireshark-essentials-15802/getting-started-15824/wireless-toolbar-15990#fn-local_id_86-2) to do channel hopping. Airodump-ng is meant to be a full-blown tool to capture wireless frames and has a handy default behavior of channel hopping without saving any data. Running sudo airodump-ng wlan0mon will achieve a result similar to the shell script above.

### Wireshark Filters

#### Display Filters

Display filters[1](https://portal.offsec.com/courses/pen-210-9545/learning/wireshark-essentials-15802/wireshark-filters-15820/wireshark-filters-15988#fn-local_id_89-1) affect which packets are visible in Wireshark's packet list. These filters impact what is displayed only, meaning that Wireshark may be capturing additional packets that are not visible.

Display filters are applied and edited in the _Filter_ toolbar located between the _Main_ toolbar and the packet list.

<figure><img src="https://static.offsec.com/offsec-courses/PEN-210/images/Wireshark_Essentials/5d3735ae9378c65d38953f8693a9ae99-ws_columns2.png" alt="Figure 10: Packet list columns"><figcaption></figcaption></figure>

The best way to understand the syntax of Wireshark display filters is to create one with the Display Filter Expression screen. Let's select _Analyze_ > _Display Filter Expression..._ to open the screen with all the available filters.

<figure><img src="https://static.offsec.com/offsec-courses/PEN-210/images/Wireshark_Essentials/2481db9036d05499adbaa2253515fc8b-ws_wireshark_dfe2.png" alt="Figure 11: Display Filter Expression builder"><figcaption></figcaption></figure>

**Packet Details**

We can also build filters based on items from a selected packet. Let's illustrate this by creating a data frame packet filter. We will start by selecting a data packet from the packet list.

<figure><img src="https://static.offsec.com/offsec-courses/PEN-210/images/Wireshark_Essentials/f6e1ecbc0fada79d021afa70293ab710-we_packet_list_data_fc2.png" alt="Figure 15: Display Filter Expression builder - Data frame"><figcaption></figcaption></figure>

**Display Filter Toolbar**

We can also create and access display filters directly in the Display Filter toolbar. On the left, a blue ribbon icon will bring us to bookmarked filters.

<figure><img src="https://static.offsec.com/offsec-courses/PEN-210/images/Wireshark_Essentials/e8f2a6e435bf69c73204ed71140fa4e0-ws_display_filter_bar2.png" alt="Figure 18: Display Filter bar"><figcaption></figcaption></figure>

To the right of the filter bar there is an arrow and a dropdown icon. The arrow will apply the filter, while the dropdown will show the most recent display filters.

<figure><img src="https://static.offsec.com/offsec-courses/PEN-210/images/Wireshark_Essentials/3610378176ad90f74cc8d95e13338677-ws_recent_df.png" alt="Figure 19: Display Filter bar - Recent filters dropdown"><figcaption></figcaption></figure>

#### Wireshark Capture Filters

_Capture Filters_[1](https://portal.offsec.com/courses/pen-210-9545/learning/wireshark-essentials-15802/wireshark-filters-15820/wireshark-display-filters-15987#fn-local_id_90-1) allow Wireshark to only collect a specific type of data. Display filters, which we just discussed, decrease the amount of data _displayed_, while capture filters limit the amount of data _received_. While they might seem similar, there are a number of reasons why we might opt to use capture filters instead of display filters.

