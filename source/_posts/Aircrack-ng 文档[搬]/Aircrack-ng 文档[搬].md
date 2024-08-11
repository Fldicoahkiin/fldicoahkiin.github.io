---
title: Aircrack-ng 文档[搬]
date: 2024-08-09 18:10:38
tags: code
---

## Airbase-ng

### Description

This documentation is still under development. There is quite a bit more work to be done on this documentation. Please post any comments or suggestions to [this thread in the Forum](http://forum.aircrack-ng.org/index.php?topic=3247.0).

Airbase-ng is a multi-purpose tool aimed at attacking clients as opposed to the Access Point (AP) itself. Since it is so versatile and flexible, summarizing it is a challenge. Here are some of the feature highlights:

- Implements the Caffe Latte WEP client attack
- Implements the Hirte WEP client attack
- Ability to cause the WPA/WPA2 handshake to be captured
- Ability to act as an ad-hoc Access Point
- Ability to act as a full Access Point
- Ability to filter by SSID or client MAC addresses
- Ability to manipulate and resend packets
- Ability to encrypt sent packets and decrypt received packets

The main idea is of the implementation is that it should encourage clients to associate with the fake AP, not prevent them from accessing the real AP.

A tap interface (atX) is created when airbase-ng is run. This can be used to receive decrypted packets or to send encrypted packets.

As real clients will most probably send probe requests for common/configured networks, these frames are important for binding a client to our softAP. In this case, the AP will respond to any probe request with a proper probe response, which tells the client to authenticate to the airbase-ng BSSID. That being said, this mode could possibly disrupt the correct functionality of many APs on the same channel.

WARNING: airbase-ng can easily disrupt Access Points around you. Where possible, use filters to minimize this possibility. Always act responsibly and do not disrupt networks which do not belong to you.

### Usage

usage: airbase-ng \<options> \<replay interface>

Options

- -a bssid : set Access Point MAC address
- -i iface : capture packets from this interface
- -w WEP key : use this WEP key to encrypt/decrypt packets
- -h MAC : source mac for MITM mode
- -f disallow : disallow specified client MACs (default: allow)
- -W 0|1 : [don't] set WEP flag in beacons 0|1 (default: auto)
- -q : quiet (do not print statistics)
- -v : verbose (print more messages) (long -''''-verbose)
- -M : M-I-T-M between [specified] clients and bssids (NOT CURRENTLY IMPLEMENTED)
- -A : Ad-Hoc Mode (allows other clients to peer) (long -''''-ad-hoc)
- -Y in|out|both : external packet processing
- -c channel : sets the channel the AP is running on
- -X : hidden ESSID (long -''''-hidden)
- -s : force shared key authentication
- -S : set shared key challenge length (default: 128)
- -L : Caffe-Latte attack (long  -''''-caffe-latte)
- -N : Hirte attack (cfrag attack), creates arp request against wep client (long --cfrag)
- -x nbpps : number of packets per second (default: 100)
- -y : disables responses to broadcast probes
- -0 : set all WPA,WEP,open tags. can't be used with -z & -Z
- -z type : sets WPA1 tags. 1=WEP40 2=TKIP 3=WRAP 4=CCMP 5=WEP104
- -Z type : same as -z, but for WPA2
- -V type : fake EAPOL 1=MD5 2=SHA1 3=auto
- -F prefix : write all sent and received frames into pcap file
- -P : respond to all probes, even when specifying ESSIDs
- -I interval : sets the beacon interval value in ms
- -C seconds : enables beaconing of probed ESSID values (requires -P)

Filter options:

- --bssid \<MAC> : BSSID to filter/use (short -b)
- --bssids \<file> : read a list of BSSIDs out of that file (short -B)
- --client \<MAC> : MAC of client to accept (short -d)
- --clients \<file> : read a list of MACs out of that file (short -D)
- --essid \<ESSID> : specify a single ESSID (short -e)
- --essids \<file> : read a list of ESSIDs out of that file (short -E)

Help:

- --help: Displays the usage screen (short -H)

#### -a BSSID Definition

If the BSSID is not explicitly specified by using "-a \<BSSID>", then the current MAC of the specified interface is used.

#### -i iface

If you specify an interface with this option then packets are also captured and processed from this interface in addition to replay interface.

#### -w WEP key

If WEP should be used as encryption, then the parameter "-w \<WEP key>" sets the en-/decryption key. This is sufficient to let airbase-ng set all the appropriate flags by itself.

If the softAP operates with WEP encryption, the client can choose to use open system authentication or shared key authentication.  Both authentication methods are supported by airbase-ng.  But to get a keystream, the user can try to force the client to use shared key authentication.  "-s" forces a shared key auth and "-S \<len>" sets the challenge length.

#### -h MAC

This is the source MAC for the man-in-the-middle attack.  The "-M" must also be specified.

#### -f allow/disallow

If this option is not specified, it defaults to "-f allow".  This means the various client MAC filters (-d and -D) define which clients to accept.

By using the "-f disallow" option, this reverses selection and causes airbase to ignore the clients specified by the filters.

#### -W WEP Flag

This sets the beacon WEP flag.  Remember that clients will normally only connect to APs which are the same as themselves.  Meaning WEP to WEP, open to open.

The "auto" option is to allow airbase-ng to automatically set the flag based on context of the other options specified.  For example, if you set a WEP key with -w, then the beacon flag would be set to WEP.

One other use of "auto" is to deal with clients which can automatically adjust their connection type.  However, these are few and far between.

In practice, it is best to set the value to the type of clients you are dealing with.

#### -q Quiet Flag

This suppresses printing any statistics or status information.

#### -v Verbose Flag

This prints additional messages and details to assist in debugging.

#### -M MITM Attack

This option is not implemented yet.  It is a man-in-the-middle attack between specified clients and BSSIDs.

#### -A Ad-Hoc Mode

This causes airbase-ng to act as an ad-hoc client instead of a normal Access Point.

In ad-hoc mode airbase-ng also sends beacons, but doesn't need any authentication/association. It can be activated by using "-A". The soft AP will adjust all flags needed to simulate a station in ad-hoc mode automatically and generate a random MAC, which is used as CELL MAC instead of the BSSID. This can be overwritten by the "-a \<BSSID>" tag. The interface MAC will then be used as source mac, which can be changed with "-h \<sourceMAC>".

#### -Y External Processing

The parameter "-Y" enables the "external processing" Mode. This creates a second interface "atX", which is used to replay/modify/drop or inject packets at will.  This interface must also be brought up with ifconfig and an external tool is needed to create a loop on that interface.

The packet structure is rather simple: the ethernet header (14 bytes) is ignored and right after that follows the complete ieee80211 frame the same way it is going to be processed by airbase-ng (for incoming packets) or before the packets will be sent out of the wireless card (outgoing packets). This mode intercepts all data packets and loops them through an external application, which decides what happens with them.  The MAC and IP of the second tap interface doesn't matter, as real ethernet frames on this interface are dropped dropped anyway.

There are 3 arguments for "-Y": "in", "out" and "both", which specify the direction of frames to loop through the external application. Obviously "in" redirects only incoming (through the wireless NIC) frames, while outgoing frames aren't touched. "out" does the opposite, it only loops outgoing packets and "both" sends all both directions through the second tap interface.

There is a small and simple example application to replay all frames on the second interface. The tool is called "replay.py" and is located in "./test". It's written in python, but the language doesn't matter.  It uses pcapy to read the frames and scapy to potentially alter/show and reinject the frames.  The tool as it is, simply replays all frames and prints a short summary of the received frames. The variable "packet" contains the complete ieee80211 packet, which can easily be dissected and modified using scapy.

This can be compared to ettercap filters, but is more powerful, as a real programming language can be used to build complex logic for filtering and packet customization. The downside on using python is, that it adds a delay of around 100ms and the cpu utilization is rather large on a high speed network, but its perfect for a demonstration with only a few lines of code.

#### -c Channel Flag

This is used to specify the channel on which to run the Access Point.

#### -X Hidden SSID Flag

This causes the Access Point to hide the SSID and to not broadcast the value.

#### -s Force Shared Key Authentication

When specified, this forces shared key authentication for all clients.

The soft AP will send an "authentication method unsupported" rejection to any open system
authentication request if "-s" is specified.

#### -S Shared Key Challenge Length

"-S \<len>" sets the challenge length, which can be anything from 16 to 1480. The default is 128 bytes.  It is the number of bytes used in the random challenge.  Since one tag can contain a maximum size of 255 bytes, any value above 255 creates several challenge tags until all specified bytes are written.  Many clients ignore values different than 128 bytes so this option may not always work.

#### -L Caffe Latte Attack

Airbase-ng also contains the new caffe-latte attack, which is also implemented in aireplay-ng as attack "-6".  It can be used with "-L" or "--caffe-latte".  This attack specifically works against clients, as it waits for a broadcast arp request, which happens to be a gratuitous arp. See [this](http://wiki.wireshark.org/Gratuitous_ARP) for an explanation of what a [gratuitous arp](http://wiki.wireshark.org/Gratuitous_ARP) is.  It then flips a few bits in the sender MAC and IP, corrects the ICV (crc32) value and sends it back to the client, where it came from.  The point why this attack works in practice is, that at least windows sends gratuitous arps after a connection on layer 2 is established and a static ip is set, or dhcp fails and windows assigned an IP out of 169.254.X.X.

"-x \<pps>" sets the number of packets per second to send when performing the caffe-latte attack. At the moment, this attack doesn't stop, it continuously sends arp requests.  Airodump-ng is needed to capture the replys.

#### -N Hirte Attack (Fragmentation Attack)

This attack listens for an ARP request or IP packet from the client.  Once one is received, a small amount of PRGA is extracted and then used to create an ARP request packet targeted to the client.  This ARP request is actually made of up of multiple packet fragments such that when received, the client will respond.

This attack works especially well against ad-hoc networks.  As well, it can be used against softAP clients and normal AP clients.

This option includes added compatibility with some clients. As well, random source IPs and MACs for cfrag attack are included to evade simple flood protection.

#### -x Number of Packets per Second

This sets the number of packets per second transmission rate (default: 100).

#### -y Disable Broadcast Probes

When using this option, the fake AP will not respond to broadcast probes.  A broadcast probe is where the specific AP is not identified uniquely.  Typically, most APs will respond with probe responses to a broadcast probe.  This flag will prevent this happening.  It will only respond when the specific AP is uniquely requested.

#### -0 Set WPA/WEP Tags

This enables all WPA/WPA2/WEP Tags to be enabled in the beacons sent.  It cannot be specified when also using -z or -Z

#### -z Set WPA Tag

This specifies the WPA beacon tags.  The valid values are: 1=WEP40 2=TKIP 3=WRAP 4=CCMP 5=WEP104.  It is recommended that you also set the WEP flag in the beacon with "-W 1" when using this parameter since some clients get confused without it.

#### -Z Set WPA2 Tag

This specifies the WPA2 beacon tags.  The valid values are the same as WPA.  It is recommended that you also set the WEP flag in the beacon with "-W 1" when using this parameter since some clients get confused without it.

#### -V EAPOL Type

This specifies the valid EAPOL types.  The valid values are: 1=MD5 2=SHA1 3=auto

#### -F File Name Prefix

This option causes airbase-ng to write all sent and received packets to a pcap file on disk. This is the file prefix (like airodump-ng -w).

#### -P All Probes

This causes the fake access point to respond to all probes regardless of the ESSIDs specified.  Without -P, the old behavior of ignoring probes for non-matching ESSIDs will be used.

#### -I Beacon Interval

This sets the time in milliseconds between beacons being sent.

When using a list of ESSIDs, all ESSIDs will be broadcast with beacons. As extra ESSIDs are added, the beacon interval value is now adjusted based on the number of ESSIDs times the interval value (0x64 is default still). To support "fast" beaconing of a long list of ESSIDs, the -I parameter can be used to set a smaller interval. To get 0x64 interval for N beacons, set the -I parameter to 0x64/N. If this value goes below ~10 or so, the maximum injection rate will be reached and airbase-ng will not be able to reliable handle new clients. Since each card's injection rates are different, the -I parameters allows it to be tuned to a specific setup and injection speed based on the number of beacons.

#### -C Seconds

The -P option must also be specified in order to use this option.  The wildcard ESSIDs will also be beaconed this number of seconds.  A good typical value to use is "-C 60".

When running in the default mode (no ESSIDs) or with the -P parameter, the -C option can be used to enable beacon broadcasting of the ESSIDs seen by the directed probes. This allows one client which is probing for a network to result in a beacon for the same network for a brief period of time (the -C parameter, which is the number of seconds to broadcast new probe requests). This works well when some clients are sending directed probes, while others listen passively for beacons. A client which does directed probes results in a beacon which wakes up the passive client and causes the passive client to join the network as well. This is especially useful with Vista clients (which listens passively for beacons in many cases) which share the same WiFi? network as Linux/Mac OS X clients which send directed probes.

#### Beacon Frames

The beacon frame contains the ESSID in case exactly one ESSID is specified, if several are set, the ESSID will be hidden in the beacon frame with a length of 1.  If no ESSID is set, the beacon will contain "default" as ESSID, but accept all ESSIDs in association requests.  If the ESSID should be hidden in the beacon frame all the time (read: for no or one specified ESSID), the "-X" flag can be set.

#### Control Frame Handling

Control frames (ack/rts/cts) are never sent by the code, but sometimes read (the firmware should handle that).  Management and data frames can always be sent, no need to authenticate before association or even sending of data frames.  They can be sent right away.  Real clients will still authenticate and associate and the softAP should send the correct answers, but airbase-ng doesn't care to check the properties and simply allows all stations to connect (with respect to the filtered ESSIDs and client MACs). So an authentication cannot fail (except if SKA is forced). Same for the association phase.  The AP will never send deauthentication or disassociation frames on normal operation mode.

It has been implemented in a way to maximizes the compatibility and the chances to keep a station connected.

#### Filtering

There are rich filtering capabilities.

To limit the supported ESSIDs, you can specify "-e \<ESSID>" to add an ESSID to the list of allowed ESSIDs, or use "-E \<ESSIDfile>" to read a list of allowed ESSIDs out of this file (one ESSID per line).

The same can be done for client MACs (sort of a MAC-filter). "-d \<MAC>" adds a single MAC to the list, "-D \<MACfile>" adds all MACs out of the \<MACfile> to that list (again, one MAC per line).

The MAC list can be used to allow only the clients on this list and block all others (default), or to block the specified ones and allow all others. This is controlled by "-f allow" or "-f disallow". "allow"
creates a whitelist (default in case "-f" is not set), whereas "disallow" a blacklist builds (the second case).

#### Tap Interface

Each time airbase is run, a tap interface (atX) is created.  To use it, run "ifconfig atX up" where X is the actual interface number.

This interface has many uses:

- If an encryption key is specified with "-w", then incoming packets will be decrypted and presented on the interface.
- Packets sent to this interface will be transmitted.  Additionally, they will be encrypted if the "-w" option is used.

### Usage Examples

Here are usage examples.  You only require a single wireless device even though two cards were used in some of the examples.

#### Simple

Using "airbase-ng \<iface>" is enough to setup an AP without any encryption. It will accept connections from any MAC for every ESSID, as long as the authentication and association is directed to the BSSID.

You really cannot do much in this scenario.  However, it will present a list of clients which are connecting plus the encryption method and the SSIDs.

#### Hirte Attack in Access Point mode

This attack obtains the wep key from a client.  It depends on receiving at least one ARP request or IP packet from the client after it has associated with the fake AP.

Enter:

``` bash
airbase-ng -c 9 -e teddy -N -W 1 rausb0
```

Where:

- -c 9 specifies the channel
- -e teddy filters a single SSID
- -N specifies the Hirte attack
- -W 1 forces the beacons to specify WEP
- rausb0 specifies the wireless interface to use

The system responds:

``` bash
18:57:54  Created tap interface at0
18:57:55  Client 00:0F:B5:AB:CB:9D associated (WEP) to ESSID: "teddy"
```

On another console window run:

``` bash
irodump-ng -c 9 -d 00:06:62:F8:1E:2C -w cfrag wlan0
```

Where:

- -c 9 specifies the channel
- -d 00:06:62:F8:1E:2C filters the data captured to fake AP MAC (this is optional)
- -w specifies the file name prefix of the captured data
- wlan0 specifies the wireless interface to capture data on

Here is what the window looks like when airbase-ng has received a packet from the client and has successfully started the attack:

``` airodump-ng
CH  9 ][ Elapsed: 8 mins ][ 2008-03-20 19:06 ]                                       
                                                                                                        
 BSSID              PWR RXQ  Beacons    #Data, #/s  CH  MB  ENC  CIPHER AUTH ESSID
                                                                                                       
 00:06:62:F8:1E:2C  100  29      970    14398   33   9  54  WEP  WEP        teddy                           
                                                                                                        
 BSSID              STATION            PWR   Rate  Lost  Packets Probes
 
 00:06:62:F8:1E:2C  00:0F:B5:AB:CB:9D   89   2-48     0   134362
```

At this point you can start aircrack-ng in another console window to obtain the wep key.  Alternatively, use the "-F \<file name prefix> option with airbase-ng to directly write a   capture file instead of using airodump-ng.

#### Hirte Attack in Ad-Hoc mode

This attack obtains the wep key from a client.  It depends on receiving at least one ARP request or IP packet from the client after it has associated with the fake AP.

Enter:

``` bash
airbase-ng -c 9 -e teddy -N -W 1 -A rausb0
```

Where:

- -c 9 specifies the channel
- -e teddy filters a single SSID
- -N specifies the Hirte attack
- -W 1 forces the beacons to specify WEP
- -A specifies ad-hoc mode
- rausb0 specifies the wireless interface to use

The rest will be the same as the AP mode.

#### Caffe Latte Attack in Access Point mode

This attack obtains the WEP key from a client.  It depends on receiving at least one gratuitous ARP request from the client after it has associated with the fake AP.

Enter:

``` bash
airbase-ng -c 9 -e teddy -L -W 1 rausb0
```

Where:

- -c 9 specifies the channel
- -e teddy filters a single SSID
- -L specifies the Caffe Latte attack
- -W 1 forces the beacons to specify WEP
- rausb0 specifies the wireless interface to use

The rest is the same as the Hirte client attack.

#### Shared Key Capture

This is an example of capturing the PRGA from a client shared key association.

Enter:

``` bash
airbase-ng -c 9 -e teddy -s -W 1 wlan0
```

Where:

- -c 9 specifies the channel
- -e teddy filters a single SSID
- -s forces shared key authentication
- -W 1 forces the beacons to specify WEP
- wlan0 specifies the wireless interface to use

The system responds:

``` airbase-ng
15:08:31  Created tap interface at0
15:13:38  Got 140 bytes keystream: 00:0F:B5:88:AC:82
15:13:38  SKA from 00:0F:B5:88:AC:82
15:13:38  Client 00:0F:B5:88:AC:82 associated to ESSID: "teddy"
```

The last three lines only appear when the client associates with the fake AP.

On another console window run:

``` bash
airodump-ng -c 9 wlan0
```

Where:

- -c 9 specifies the channel
- wlan0 specifies the wireless interface to use

Here is what the window looks like with a successful SKA capture.  Notice "140 bytes keystream: 00:C0:CA:19:F9:65" in the top right-hand corner:

``` airodump-ng
CH  9 ][ Elapsed: 9 mins ][ 2008-03-12 15:13 ][ 140 bytes keystream: 00:C0:CA:19:F9:65 ]

 BSSID              PWR RXQ  Beacons    #Data, #/s  CH  MB  ENC  CIPHER AUTH ESSID

 00:C0:CA:19:F9:65   87  92     5310        0    0   9  54  WEP  WEP    SKA  teddy

 BSSID              STATION            PWR   Rate  Lost  Packets  Probes

 00:C0:CA:19:F9:65  00:0F:B5:88:AC:82   83   0- 1     0     4096  teddy 
```

Alternatively, use the "-F \<file name prefix> option with airbase-ng to directly write a   capture file instead of using airodump-ng.

#### WPA Handshake Capture

This is an example of how to capture the WPA handshake.

Enter:

``` bash
airbase-ng -c 9 -e teddy -z 2 -W 1 rausb0
```

Where:

- -c 9 specifies the channel
- -e teddy filters a single SSID
- -z 2 specifies TKIP
- -W 1 set WEP flag because some clients get confused without it.
- rausb0 specifies the wireless interface to use

The -z type will have to be changed depending on the cipher you believe the client will be using.  TKIP is typical for WPA.

The system responds:

``` airbase-ng
10:17:24  Created tap interface at0
10:22:13  Client 00:0F:B5:AB:CB:9D associated (WPA1;TKIP) to ESSID: "teddy"
```

The last line only appears when the client associates.

On another console window run:

``` bash
airodump-ng -c 9 -d 00:C0:C6:94:F4:87 -w cfrag wlan0
```

- -c 9 specifies the channel
- -d 00:C0:C6:94:F4:87 filters the data captured to fake AP MAC.  It is MAC of card running the fake AP.  This is optional.
- -w specifies the file name of the captured data
- wlan0 specifies the wireless interface to capture data on

When the client connects, notice the "WPA handshake: 00:C0:C6:94:F4:87" in the top right-hand corner of the screen below:

``` airodump-ng
   CH  9 ][ Elapsed: 5 mins ][ 2008-03-21 10:26 ][ WPA handshake: 00:C0:C6:94:F4:87 ]

   BSSID              PWR RXQ  Beacons    #Data, #/s  CH  MB  ENC  CIPHER AUTH ESSID

   00:C0:C6:94:F4:87  100  70     1602       14    0   9  54  WPA  TKIP   PSK  teddy
  
   BSSID              STATION            PWR   Rate  Lost  Packets  Probes
  
   00:C0:C6:94:F4:87  00:0F:B5:AB:CB:9D   86   2- 1     0       75
```

Alternatively, use the "-F \<file name prefix> option with airbase-ng to directly write a   capture file instead of using airodump-ng.

Running "aircrack-ng cfrag-01.cap" proves it captured a valid WPA handshake:

``` aircrack-ng
 Opening cfrag-01.cap
 Read 114392 packets.
 
 #  BSSID              ESSID                     Encryption
 
 1  00:C0:C6:94:F4:87  teddy                     WPA (1 handshake)
```

#### WPA2 Handshake Capture

Capturing a WPA2 is basically identical to the example above regarding WPA.  The typical difference is to specify -Z 4 (CCMP cipher) instead of -z 2.

Enter:

``` bash
airbase-ng -c 9 -e teddy -Z 4 -W 1 rausb0
```

The balance is the same as the WPA handshake capture.

#### softAP

GURU EXPERTS ONLY:  This functionality requires extremely advanced linux and networking knowledge.  Do not post questions to the forum regarding this section.  If you cannot debug this functionality on your own then you should not be using it!

A new tap interface "atX" will be created, which acts as the "wired side" to the AP. In order to use the AP, this new interface must be brought up with ifconfig and needs an IP. The assigned MAC is automatically set to the BSSID [by default the wireless interface MAC]. Once an IP is assigned and the client uses a static IP out of the same subnet, there is a working Ethernet connection between the AP and the client. Any daemon can be assigned to that interface, for example a dhcp and dns server. Together with kernel ip_forwarding and a proper iptable rule for masquerading, the softAP acts as a wireless router. Any tool, which operates on ethernet can be bound to this interface.

This [forum posting](http://forum.aircrack-ng.org/index.php?topic=3983.msg23110#msg23110) provides an example of the commands needed to setup the softAP. This [forum posting](http://forum.aircrack-ng.org/index.php?topic=4495.msg25342#msg25342) provides IPTables troubleshooting tip.

Here are some links that may find useful in getting bridging operational.  In the madwifi-project.org one, just use at0 where ath0 is referenced.

- <http://madwifi-project.org/wiki/UserDocs/TransparentBridge>
- <http://wiki.ubuntuusers.de/WLAN/MadWifi#Einfache-Methode>

### Usage Tips

#### How Does the Caffe Latte Attack Work?

This is a client attack which can use any IP or ARP packet. The following describes the attack in detail.

The basic idea is to generate an ARP request to be sent back to the client such that the client responds.

The attack needs either an ARP or IP packet from the client. From this, we need to generate an ARP request. The ARP request must have the target IP (client IP) at byte position 33 and the target MAC should be all zeroes. However the target MAC can really be any value in practice.

The source IP is in the packet received from the client is in a known position - position 23 for ARP or 21 for IP. ARP is assumed if the packet is 68 or 86 bytes in length plus a broadcast destination MAC address. Otherwise it is assumed to be an IP packet.

In order to send a valid ARP request back to the client, we need to move the source IP to position 33. Of course you can't simply move bytes around, that would invalidate the packet. So instead, we use the concept of packet fragmentation to achieve this. The ARP request is sent to the client as two fragments. The first fragment length is selected such that the incoming source IP is moved to position 33 when the fragments are ultimately reassembled by the client. The second fragment is the original packet received from the client.

In the case of an IP packet, a similar technique is used. However due to the more limited amount of PRGA available, there are three fragments plus the original packet used.

In all cases, bit flipping is used to ensure the CRC is correct. Additionally, bit flipping is used to ensure the source MAC of the ARP contained within the fragmented packet is not multicast.

#### SoftAP with Internet connection and MITM sniffing

This [forum thread](http://forum.aircrack-ng.org/index.php?topic=7172.0) provides a tutorial for SoftAP with Internet connection and MITM sniffing.

### Usage Troubleshooting

#### Driver Limitations

Some drivers like r8187 don't capture packets transmitted by itself.  The implication of this is that the softAP will not show up in airodump-ng.  You can get around this by using two wireless cards, one to inject and one to capture. Alternatively, you can use the rtl8187 driver.

The madwifi-ng currently does not support the Caffe-Latte or Hirte attacks.  The root cause is deep within the madwifi-ng driver.  The driver does not properly synchronize speeds with the client and thus the client never receives the packets. If you need to use these attacks, try using the ath5k driver.

#### Broken SKA error message

You receive "Broken SKA: \<MAC address> (expected: ??, got ?? bytes)" or similar. When using the "-S" option with values different then 128, some clients fail.  This message indicates the number of bytes actually received was different that the number requested.  Either don't use the option or try different values of "-S" to see which one eliminates the error.

#### "write failed: Message too long" / "wi_write(): Illegal seek" error messages

See this [GitHub issue](https://github.com/aircrack-ng/aircrack-ng/issues/469) for a workaround.  The issue explains the root cause and how to adjust the MTU to avoid the problem.

#### Error creating tap interface: Permission denied

See the following [[faq#why_do_i_get_error_creating_tap_interfacepermission_denied_or_a_similar_message|FAQ entry]].

### Related Commands

"-D" is a new option that has been added to aireplay-ng.  By default, aireplay-ng listens for beacons from the specified AP and fails if it does not hear any beacons.  The "-D" option disables this requirement.

#### aireplay-ng -6  (Cafe Latte attack)

Example: aireplay-ng -6  -h 00:0E:D2:8D:7D:0A  -D  rausb0

#### aireplay-ng -7  (Hirte attack)

Example: aireplay-ng -7  -h 00:0E:D2:8D:7D:0A  -D  rausb0

## Aircrack-ng

### Description

Aircrack-ng is an 802.11 WEP and WPA/WPA2-PSK key cracking program.

Aircrack-ng can recover the WEP key once enough encrypted packets have been captured with [[airodump-ng]]. This part of the aircrack-ng suite determines the WEP key using two fundamental methods.  The first method is via the PTW approach (Pyshkin, Tews, Weinmann).  The default cracking method is PTW.  This is done in two phases.  In the first phase, aircrack-ng only uses ARP packets.  If the key is not found, then it uses all the packets in the capture.  Please remember that not all packets can be used for the PTW method.  This [[supported_packets|Tutorial: Packets Supported for the PTW Attack page]] provides details.  An important limitation is that the PTW attack currently can only crack 40 and 104 bit WEP keys. The main advantage of the PTW approach is that very few data packets are required to crack the WEP key.

The other, older method is the FMS/KoreK method.  The FMS/KoreK method incorporates various statistical attacks to discover the WEP key and uses these in combination with brute forcing. It requires more packets than PTW, but on the other hand is able to recover the passphrase when PTW sometimes fail.

Additionally, the program offers a dictionary method for determining the WEP key.

For cracking WPA/WPA2 pre-shared keys, only a dictionary method is used.  A "four-way handshake" is required as input.  For WPA handshakes, a full handshake is composed of four packets.  However, aircrack-ng is able to work successfully with just 2 packets.  EAPOL packets (2 and 3) or packets (3 and 4) are considered a full handshake.

SSE2, AVX, AVX2, and AVX512 support is included to dramatically speed up WPA/WPA2 key processing. With the exception of AVX512, all other instructions are built-in Aircrack-ng, and it will automatically select the fastest available for the CPU. For non-x86 CPUs, SIMD improvements are present as well.

#### Screenshot

*LEGEND*

1 = Keybyte  
2 = Depth of current key search  
3 = Byte the IVs leaked  
4 = Votes indicating this is correct

![aircrack-ng.explaination.gif](aircrack-ng.explaination.gif?600x300)

#### How does it work?

The first method is the PTW method (Pychkine, Tews, Weinmann). The PTW method is fully described in the paper found on  [this web site](https://web.archive.org/web/20070714194826/http://www.cdc.informatik.tu-darmstadt.de/aircrack-ptw/).  In 2005, Andreas Klein presented another analysis of the RC4 stream cipher. Klein showed that there are more correlations between the RC4 keystream and the key than the ones found by Fluhrer, Mantin, and Shamir and these may be additionally used to break WEP.  The PTW method extends Klein's attack and optimizes it for usage against WEP.  It essentially uses enhanced FMS techniques described in the following section.  One particularly important constraint is that it only works with arp request/reply packets and cannot be employed against other traffic.

The second method is the FMS/Korek method which incorporates multiple techniques.  The  [[links#technique_papers|Techniques Papers]] on the links page lists many papers which describe these techniques in more detail and the mathematics behind them.

In this method, multiple techniques are combined to crack the WEP key:

- FMS (Fluhrer, Mantin, Shamir) attacks - statistical techniques
- Korek attacks - statistical techniques
- Brute force

When using statistical techniques to crack a WEP key, each byte of the key is essentially handled individually.  Using statistical mathematics, the possibility that a certain byte in the key is correctly guessed goes up to as much as 15% when the right initialization vector (IV) is captured for a particular key byte.  Essentially, certain IVs "leak" the secret WEP key for particular key bytes.  This is the fundamental basis of the statistical techniques.

By using a series of statistical tests called the FMS and Korek attacks, votes are accumulated for likely keys for each key byte of the secret WEP key.  Different attacks have a different number of votes associated with them since the probability of each attack yielding the right answer varies mathematically.  The more votes a particular potential key value accumulates, the more likely it is to be correct.  For each key byte, the screen shows the likely secret key and the number of votes it has accumulated so far.  Needless to say, the secret key with the largest number of votes is most likely correct but is not guaranteed.  Aircrack-ng will subsequently test the key to confirm it.

Looking at an example will hopefully make this clearer.  In the screenshot above, you can see, that at key byte 0 the byte 0xAE has collected some votes, 50 in this case.  So, mathematically, it is more likely that the key starts with AE than with 11 (which is second on the same line) which is almost half as possible.  That explains why the more data that is available, the greater the chances that aircrack-ng will determine the secret WEP key.

However the statistical approach can only take you so far.  The idea is to get into the ball park with statistics then use brute force to finish the job.  Aircrack-ng uses brute force on likely keys to actually determine the secret WEP key.

This is where the fudge factor comes in.  Basically the fudge factor tells aircrack-ng how broadly to brute force.  It is like throwing a ball into a field then telling somebody to ball is somewhere between 0 and 10 meters (0 and 30 feet) away.  Versus saying the ball is somewhere between 0 and 100 meters (0 and 300 feet) away.  The 100 meter scenario will take a lot longer to search then the 10 meter one but you are more likely to find the ball with the broader search.  It is a trade off between the length of time and likelihood of finding the secret WEP key.

For example, if you tell aircrack-ng to use a fudge factor 2, it takes the votes of the most possible byte, and checks all other possibilities which are at least half as possible as this one on a brute force basis.  The larger the fudge factor, the more possibilities aircrack-ng will try on a brute force basis.  Keep in mind, that as the fudge factor gets larger, the number of secret keys to try goes up tremendously and consequently the elapsed time also increases.  Therefore with more available data, the need to brute force, which is very CPU and time intensive, can be minimized.

In the end, it is all just "simple" mathematics and brute force!

For cracking WEP keys, a dictionary method is also included.  For WEP, you may use either the statistical method described above or the dictionary method, not both at the same time.  With the dictionary method, you first create a file with either ascii or hexadecimal keys.  A single file can only contain one type, not a mix of both.  This is then used as input to aircrack-ng and the program tests each key to determine if it is correct.

The techniques and the approach above do not work for WPA/WPA2 pre-shared keys.  The only way to crack these pre-shared keys is via a dictionary attack.  This capability is also included in aircrack-ng.

With pre-shared keys, the client and access point establish keying material to be used for their communication at the outset, when the client first associates with the access point.  There is a four-way handshake between the client and access point.  [airodump-ng](https://www.aircrack-ng.org/doku.php?id=airodump-ng) can capture this four-way handshake.  Using input from a provided word list (dictionary), aircrack-ng duplicates the four-way handshake to determine if a particular entry in the word list matches the results the four-way handshake.  If it does, then the pre-shared key has been successfully identified.

It should be noted that this process is very computationally intensive and so in practice, very long or unusual pre-shared keys are unlikely to be determined.  A good quality word list will give you the best results.  Another approach is to use a tool like john the ripper to generate password guesses which are in turn fed into aircrack-ng.

#### Explanation of the Depth Field and Fudge Factor

The best explanation is an example.  We will look at a specific byte.  All bytes are processed in the same manner.

You have the votes like in the screen shot above.  For the first byte they look like:
AE(50) 11(20) 71(20) 10(12) 84(12)

The AE, 11, 71, 10 and 84 are the possible secret key for key byte 0.  The numbers in parentheses are the votes each possible secret key has accumulated so far.

Now if you decide to use a fudge factor of 3.  Aircrack-ng takes the vote from the most possible byte AE(50):

50 / 3 = 16.666666

Aircrack-ng will test (brute force) all possible keys with a vote greater than 16.6666, resulting in

AE, 11, 71

being tested, so we have a total depth of three:

0 / 3     AE(50) 11(20) 71(20) 10(12) 84(12)

When aircrack-ng is testing keys with AE, it shows 0 / 3, if it has all keys tested with that byte, it switches to the next one (11 in this case) and displays:

1 / 3     11(20) 71(20) 10(12) 84(12)

### Usage

``` bash
aircrack-ng [options] <capture file(s)>
```

You can specify multiple input files (either in .cap or .ivs format) or use file name wildcarding.  See [[aircrack-ng#other_tips|Other Tips]] for examples.  Also, you can run both [airodump-ng](https://www.aircrack-ng.org/doku.php?id=airodump-ng) and aircrack-ng at the same time: aircrack-ng will auto-update when new IVs are available.

#### Options

##### Common options

|Option|Param.|Description|
|-|-|-|
|-a|amode|Force attack mode (1 = static WEP, 2 = WPA/WPA2-PSK)|
|-e|essid|If set, all IVs from networks with the same ESSID will be used. This option is also required for WPA/WPA2-PSK cracking if the ESSID is not broadcasted (hidden)|
|-b|bssid|Long version -''''-bssid. Select the target network based on the access point's MAC address|
|-p|nbcpu|On SMP systems: # of CPU to use.  This option is invalid on non-SMP systems|
|-q|*none*|Enable quiet mode (no status output until the key is found, or not)|
|-C|MACs|Long version -''''-combine.  Merge the given APs (separated by a comma) into virtual one|
|-l|file name|(Lowercase L, ell) logs the key to the file specified. Overwrites the file if it already exists|

##### Static WEP cracking options

|Option|Param.|Description|
|-|-|-|
|-c|*none*|Restrict the search space to alpha-numeric characters only (0x20 - 0x7F)|
|-t|*none*|Restrict the search space to binary coded decimal hex characters|
|-h|*none*|Restrict the search space to numeric characters (0x30-0x39) These keys are used by default in most Fritz!BOXes|
|-d|start|Long version -''''-debug.  Set the beginning of the WEP key (in hex), for debugging purposes|
|-m|maddr|MAC address to filter WEP data packets. Alternatively, specify -m ff:ff:ff:ff:ff:ff to use all and every IVs, regardless of the network|
|-n|nbits|Specify the length of the key: 64 for 40-bit WEP, 128 for 104-bit WEP, etc. The default value is 128|
|-i|index|Only keep the IVs that have this key index (1 to 4). The default behaviour is to ignore the key index|
|-f|fudge|By default, this parameter is set to 2 for 104-bit WEP and to 5 for 40-bit WEP. Specify a higher value to increase the bruteforce level: cracking will take more time, but with a higher likelyhood of success|
|-k|korek|There are 17 korek statistical attacks. Sometimes one attack creates a huge false positive that prevents the key from being found, even with lots of IVs. Try -k 1, -k 2, ... -k 17 to disable each attack selectively|
|-x/-x0|*none*|Disable last keybytes brutforce|
|-x1|*none*|Enable last keybyte bruteforcing (default)|
|-x2|*none*|Enable last two keybytes bruteforcing|
|-X|*none*|Disable bruteforce multithreading (SMP only)|
|-s|*none*|Show the key in ASCII while cracking|
|-y|*none*|Experimental single bruteforce attack which should only be used when the standard attack mode fails with more than one million IVs|
|-z|*none*|Invokes the PTW WEP cracking method (Default in v1.x)|
|-P|number|Long version -''''-ptw-debug.  Invokes the PTW debug mode: 1 Disable klein, 2 PTW.|
|-K|*none*|Invokes the Korek WEP cracking method. (Default in v0.x)|
|-D|*none*|Long version -''''-wep-decloak.  Run in WEP decloak mode|
|-1|*none*|Long version -''''-oneshot.  Run only 1 try to crack key with PTW|
|-M|number|(WEP cracking) Specify the maximum number of IVs to use|
|-V|*none*|Long version -''''-visual-inspection.  Run in visual inspection mode (only with KoreK)|

##### WEP and WPA-PSK cracking options

|Option|Param.|Description|
|-|-|-|
|-w|words|Path to a wordlists or "-" without the quotes for standard in (stdin). Separate multiple wordlists by comma|
|-N|file|Create a new cracking session and save it to the specified file|
|-R|file|Restore cracking session from the specified file|

##### WPA-PSK options

|Option|Param.|Description|
|-|-|-|
|-E|file|Create EWSA Project file v3|
|-j|file|Create Hashcat v3.6+ Capture file (HCCAPX)|
|-J|file|Create Hashcat Capture file|
|-S|*none*|WPA cracking speed test|
|-Z|sec|WPA cracking speed test execution length in seconds|
|-r|database|Utilizes a database generated by [airolib-ng](https://www.aircrack-ng.org/doku.php?id=airolib-ng) as input to determine the WPA key. Outputs an error message if aircrack-ng has not been compiled with sqlite support|

##### SIMD Selection

|Option|Param.|Description|
|-|-|-|
|--simd|optimization|Use user-specified SIMD optimization instead of the fastest one|
|--simd-list|*none*|Shows a list of the SIMD optimizations available|

##### Other options

|Option|Param.|Description|
|-|-|-|
|-H|*none*|Long version --help.  Output help information|
|-u|*none*|Long form --cpu-detect.  Provide information on the number of CPUs and features available such as MMX, SSE2, AVX, AVX2, AVX512|

### Usage Examples

#### WEP

The simplest case is to crack a WEP key. If you want to try this out yourself, here is a test [file](https://download.aircrack-ng.org/wiki-files/other/test.ivs).  The key to the test file matches the screen image above, it does not match the following example.

aircrack-ng -K 128bit.ivs  
Where:

- 128bit.ivs is the file name containing IVS.
- -K: Use KoreK attacks only

The program responds:

``` aircrack-ng
Opening 128bit.ivs
Read 684002 packets.

#  BSSID              ESSID                     Encryption

1  00:14:6C:04:57:9B                            WEP (684002 IVs)

Choosing first network as target.
```

If there were multiple networks contained in the file then you are given the option to select which one you want. By default, aircrack-ng assumes 128 bit encryption.

The cracking process starts and once cracked, here is what it looks like:

``` aircrack-ng
                                              Aircrack-ng 1.4


                              [00:00:10] Tested 77 keys (got 684002 IVs)

 KB    depth   byte(vote)
  0    0/  1   AE( 199) 29(  27) 2D(  13) 7C(  12) FE(  12) FF(   6) 39(   5) 2C(   3) 00(   0) 08(   0) 
  1    0/  3   66(  41) F1(  33) 4C(  23) 00(  19) 9F(  19) C7(  18) 64(   9) 7A(   9) 7B(   9) F6(   9) 
  2    0/  2   5C(  89) 52(  60) E3(  22) 10(  20) F3(  18) 8B(  15) 8E(  15) 14(  13) D2(  11) 47(  10) 
  3    0/  1   FD( 375) 81(  40) 1D(  26) 99(  26) D2(  23) 33(  20) 2C(  19) 05(  17) 0B(  17) 35(  17) 
  4    0/  2   24( 130) 87( 110) 7B(  32) 4F(  25) D7(  20) F4(  18) 17(  15) 8A(  15) CE(  15) E1(  15) 
  5    0/  1   E3( 222) 4F(  46) 40(  45) 7F(  28) DB(  27) E0(  27) 5B(  25) 71(  25) 8A(  25) 65(  23) 
  6    0/  1   92( 208) 63(  58) 54(  51) 64(  35) 51(  26) 53(  25) 75(  20) 0E(  18) 7D(  18) D9(  18) 
  7    0/  1   A9( 220) B8(  51) 4B(  41) 1B(  39) 3B(  23) 9B(  23) FA(  23) 63(  22) 2D(  19) 1A(  17) 
  8    0/  1   14(1106) C1( 118) 04(  41) 13(  30) 43(  28) 99(  25) 79(  20) B1(  17) 86(  15) 97(  15) 
  9    0/  1   39( 540) 08(  95) E4(  87) E2(  79) E5(  59) 0A(  44) CC(  35) 02(  32) C7(  31) 6C(  30) 
 10    0/  1   D4( 372) 9E(  68) A0(  64) 9F(  55) DB(  51) 38(  40) 9D(  40) 52(  39) A1(  38) 54(  36) 
 11    0/  1   27( 334) BC(  58) F1(  44) BE(  42) 79(  39) 3B(  37) E1(  34) E2(  34) 31(  33) BF(  33) 

           KEY FOUND! [ AE:66:5C:FD:24:E3:92:A9:14:39:D4:27:4B ] 
```

**NOTE:** The ASCII WEP key is displayed only when 100% of the hex key can be converted to ASCII.

This key can then be used to connect to the network.

Next, we look at cracking WEP with a dictionary.  In order to do this, we need dictionary files with ascii or hexadecimal keys to try.  Remember, a single file can only have ascii or hexadecimal keys in it, not both.

WEP keys can be entered in hexadecimal or ascii.  The following table describes how many characters of each type is required in your files.

|WEP key length in bits|Hexadecimal Characters|Ascii Characters|
|-|-|-|
|64|10|5|
|128|26|13|
|152|32|16|
|256|58|29|

Example 64 bit ascii key: “ABCDE”  
Example 64 bit hexadecimal key: “12:34:56:78:90” (Note the “:” between each two characters.)  
Example 128 bit ascii key: “ABCDEABCDEABC”  
Example 128 bit hexadecimal key: “12:34:56:78:90:12:34:56:78:90:12:34:56”
To WEP dictionary crack a 64 bit key:

aircrack-ng -w h:hex.txt,ascii.txt -a 1 -n 64 -e teddy wep10-01.cap

Where:

- -w h:hex.txt,ascii.txt is the list of files to use. For files containing hexadecimal values, you must put a “h:” in front of the file name.
- -a 1 says that it is WEP
- -n 64 says it is 64 bits. Change this to the key length that matches your dictionary files.
- -e teddy is to optionally select the access point. Your could also use the “-b” option to select based on MAC address
- wep10-01.cap is the name of the file containing the data. It can be the full packet or an IVs only file. It must contain be a minimum of four IVs.

Here is a sample of the output:

``` aircrack-ng
                                              Aircrack-ng 1.4
 
 
                              [00:00:00] Tested 2 keys (got 13 IVs)
 
 KB    depth   byte(vote)
  0    0/  0   00(   0) 00(   0) 00(   0) 00(   0) 00(   0) 00(   0) 00(   0) 00(   0) 00(   0) 00(   0) 
  1    0/  0   00(   0) 00(   0) 00(   0) 00(   0) 00(   0) 00(   0) 00(   0) 00(   0) 00(   0) 00(   0) 
  2    0/  0   00(   0) 00(   0) 00(   0) 00(   0) 00(   0) 00(   0) 00(   0) 00(   0) 00(   0) 00(   0) 
  3    0/  0   00(   0) 00(   0) 00(   0) 00(   0) 00(   0) 00(   0) 00(   0) 00(   0) 00(   0) 00(   0) 
  4    0/  0   00(   0) 00(   0) 00(   0) 00(   0) 00(   0) 00(   0) 00(   0) 00(   0) 00(   0) 00(   0) 
 
                       KEY FOUND! [ 12:34:56:78:90 ] 
      Probability: 100%
```

Lets look at a PTW attack example. Remember that this method requires arp request/reply packets as input. It must be the full packet and not just the IVs, meaning that the “-- ivs” option cannot be used when running airodump-ng. As well, it only works for 64 and 128 bit WEP encryption.

Enter the following command:

``` bash
aircrack-ng -z ptw*.cap  
```

Where:

- -z means use the PTW methodology to crack the wep key. Note: in v1.x, this is the default attack mode; use -K to revert to Korek.
- ptw*.cap are the capture files to use.

The systems responds:

``` aircrack-ng
 Opening ptw-01.cap
 Read 171721 packets.
    
 #  BSSID              ESSID                     Encryption
 
 1  00:14:6C:7E:40:80  teddy                     WEP (30680 IVs)
 
 Choosing first network as target.
```

Then:

``` aircrack-ng
                                              Aircrack-ng 1.4
 
                              [00:01:18] Tested 0/140000 keys (got 30680 IVs)
 
 KB    depth   byte(vote)
  0    0/  1   12( 170) 35( 152) AA( 146) 17( 145) 86( 143) F0( 143) AE( 142) C5( 142) D4( 142) 50( 140) 
  1    0/  1   34( 163) BB( 160) CF( 147) 59( 146) 39( 143) 47( 142) 42( 139) 3D( 137) 7F( 137) 18( 136) 
  2    0/  1   56( 162) E9( 147) 1E( 146) 32( 146) 6E( 145) 79( 143) E7( 142) EB( 142) 75( 141) 31( 140) 
  3    0/  1   78( 158) 13( 156) 01( 152) 5F( 151) 28( 149) 59( 145) FC( 145) 7E( 143) 76( 142) 92( 142) 
  4    0/  1   90( 183) 8B( 156) D7( 148) E0( 146) 18( 145) 33( 145) 96( 144) 2B( 143) 88( 143) 41( 141) 
 
                       KEY FOUND! [ 12:34:56:78:90 ] 
      Decrypted correctly: 100%
```

#### WPA

Now onto cracking WPA/WPA2 passphrases.  Aircrack-ng can crack either types.

aircrack-ng -w password.lst *.cap  
Where:

- -w password.lst is the name of the password file. Remember to specify the full path if the file is not located in the same directory.
- \*.cap is name of group of files containing the captured packets. Notice in this case that we used the wildcard * to include multiple files.

The program responds:

``` aircrack-ng
Opening wpa2.eapol.cap
Opening wpa.cap
Read 18 packets.

#  BSSID              ESSID                     Encryption

1  00:14:6C:7E:40:80  Harkonen                  WPA (1 handshake)
2  00:0D:93:EB:B0:8C  test                      WPA (1 handshake)

Index number of target network ? 
```

Notice in this case that since there are multiple networks we need to select which one to attack. We select number 2. The program then responds:

``` aircrack-ng
                               Aircrack-ng 1.4


                 [00:00:03] 230 keys tested (73.41 k/s)


                         KEY FOUND! [ biscotte ]


    Master Key     : CD D7 9A 5A CF B0 70 C7 E9 D1 02 3B 87 02 85 D6 
                     39 E4 30 B3 2F 31 AA 37 AC 82 5A 55 B5 55 24 EE 

    Transcient Key : 33 55 0B FC 4F 24 84 F4 9A 38 B3 D0 89 83 D2 49 
                     73 F9 DE 89 67 A6 6D 2B 8E 46 2C 07 47 6A CE 08 
                     AD FB 65 D6 13 A9 9F 2C 65 E4 A6 08 F2 5A 67 97 
                     D9 6F 76 5B 8C D3 DF 13 2F BC DA 6A 6E D9 62 CD 

    EAPOL HMAC     : 52 27 B8 3F 73 7C 45 A0 05 97 69 5C 30 78 60 BD 
```

Now you have the passphrase and can connect to the network.

##### SIMD

Aircrack-ng is compiled with multiple optimizations based on CPU features we call crypto engines. CPU features are different based on the type of CPU.

On x86 (and 64 bit), typically SSE2, AVX and AVX2 are available (AVX512 can be compiled in but it should only be done if the current CPU supports it). On ARM, neon and ASIMD are usually available and on PowerPC, ASIMD and altivec. A generic optimization is always available no matter what architecture it is compiled on or for. A limited set of optimizations may be available depending on the OS/CPU/compilers available.

When running aircrack-ng, it will load the fastest optimization based on what your CPU supports. For package maintainers, it is very useful as they don't have to target the one supporting all the CPU which would be the slowest.

In order to override, the option --simd can be used. Such as

``` bash
aircrack-ng --simd=avx wpa.cap -w password.lst
```

In order to list all the available SIMD optimization, use --simd-list. Such as

``` bash
aircrack-ng --simd-list
```

will display “avx2 avx sse2 generic” on x86.

#### Cracking session

Cracking can sometimes take a very long time and it is sometimes necessary to turn off the computer or put it to sleep for a while. In order to handle this kind of situation, a new set of option has been created.

It will create and/or update a session file saving the current status of the cracking (every 10 minutes) as well as all the options used, wordlists and capture files used. Multiple wordlists can be used and it works with WEP and WPA.

``` bash
aircrack-ng --new-session current.session -w password.lst,english.txt wpa-01.cap 
```

In order to restore the session, use --restore-session:

``` bash
aircrack-ng --restore-session current.session
```

It will keep updating current.session every 10 minutes.

Limitations:

- The wordlist must be files. For now, they cannot be stdin or airolib-ng databases
- Session has to be restored from the same directory as when first using --new-session
- No new options can be added when restoring session

### Usage Tips

#### General approach to cracking WEP keys

*FIXME This needs updating for v1.x!*

Clearly, the simplest approach is just to enter "aircrack-ng captured-data.cap" and let it go.  Having said that, there are some techniques to improve your chances of finding the WEP key quickly.  There is no single magic set of steps.  The following describes some approaches which tend to  yield the key faster.  Unless you are comfortable with experimentation, leave well enough alone and stick to the simple approach.

If you are capturing arp request/reply packets, then the fastest approach is to use "aircrack-ng -z \<data packet capture files>".  You can then skip the balance of this section since it will find the key very quickly assuming you have collected sufficient arp request/reply packets! //NOTE:// -z is the default attack mode in aircrack-ng v1.x; use -K to revert to the attack mode used in previous versions.

The overriding technique is capture as much data as possible.  That is the single most important task.  The number of initialization vectors (IVs) that you need to determine the WEP key varies dramatically by key length and access point.  Typically you need 250,000 or more unique IVs for 64 bit keys and 1.5 million or more for 128 bit keys.  Clearly a lot more for longer key bit lengths.  Then there is luck.  There will be times that the WEP key can be determined with as few as 50,000 IVs although this is rare.  Conversely, there will be times when you will need mulitple millions of IVs to crack the WEP key.  The number of IVs is extremely hard to predict since some access points are very good at eliminating IVs that lead the WEP key.

Generally, don't try to crack the WEP key until you have 200,000 IVs or more.  If you start too early, aircrack tends to spend too much time brute forcing keys and not properly applying the statistical techniques.  Start by trying 64 bit keys "aircrack-ng -n 64 captured-data.cap".  If they are using a 64 bit WEP, it can usually be cracked in less then 5 minutes (generally less then 60 seconds) with relatively few IVs.  It is surprising how many APs only use 64 bit keys.  If it does not find the 64 bit key in 5 minutes, restart aircrack in the generic mode: "aircrack-ng captured-data.cap".  Then at each 100,000 IVs mark, retry the "aircrack-ng -n 64 captured-data.cap" for 5 minutes.

Once you hit 600,000 IVs, switch to testing 128 bit keys.  At this point it is unlikely (but not impossible) that it is a 64 bit key and 600,000 IVs did not crack it.  So now try "aircrack-ng captured-data.cap".

Once you hit 2 million IVs, try changing the fudge factor to "-f 4".  Run for at least 30 minutes to one hour.  Retry, increasing the fudge factor by adding 4 to it each time.  Another time to try increasing the fudge factor is when aircrack-ng stops because it has tried all the keys.

All the while, keep collecting data.  Remember the golden rule, "the more IVs the better".

Also check out the next section on how to determine which options to use as these can significantly speed up cracking the WEP key.  For example, if the key is all numeric, then it can take as few as 50,000 IVs to crack a 64 bit key with the "-t" versus 200,000 IVs without the "-t".  So if you have a hunch about the nature of the WEP key, it is worth trying a few variations.

#### How to determine which options to use

While aircrack-ng is running, you mostly just see the beginning of the key.  Although the secret WEP key is unknown at this point, there may be clues to speed things up.  If the key bytes have a fairly large number of votes, then they are likely 99.5% correct.  So lets look at what you can do with these clues.

If the bytes (likely secret keys) are for example: 75:47:99:22:50 then it is quite obvious, that the whole key may consist only of numbers, like the first 5 bytes.  So it MAY improve your cracking speed to use the -t option only when trying such keys.  See [Wikipedia Binary Coded Decimal](https://en.wikipedia.org/wiki/Binary-coded_decimal) for a description of what characters -t looks for.

If the bytes are 37:30:31:33:36 which are all numeric values when converted to Ascii, it is a good idea to use -h option.  The FAQ entry [Converting hex characters to ascii](https://www.aircrack-ng.org/doku.php?id=faq#how_do_i_convert_the_hex_characters_to_ascii) provides links to determine if they are all numeric.

And if the first few bytes are something like 74:6F:70:73:65, and upon entering them into your hexeditor or the links provided in the previous sentence, you see that they may form the beginning of some word, then it seems likely an ASCII key is used, thus you activate -c option to check only printable ASCII keys.

If you know the start of the WEP key in hexadecimal, you can enter with the "-d" parameter.  Lets assume you know the WEP key is "0123456789" in hexadecimal then you could use "-d 01" or "-d 0123", etc.

Another option to try when having problems determining the WEP key, is the "-x2" option which causes the last two keybytes to be brute forced instead of the default of one.

#### How to convert the HEX WEP key to ASCII?

See the next entry.

#### How to use the key

If aircrack-ng determines the key, it is presented to you in hexadecimal format. It typically looks like:

``` aircrack-ng
KEY FOUND! [11:22:33:44:55]
```

The length will vary based on the WEP bit key length used.  See the table above which indicates the number of hexadecimal characters for the various WEP key bit lenghts.

You may use this key without the ":" in your favorite client.  This means you enter "1122334455" into the client and specify that the key is in hexadecimal format.  Remember that most keys cannot be converted to ASCII format.  If the HEX key is in fact valid ASCII characters, the ASCII will also be displayed.

If you wish to experiment a bit with converting HEX to ASCII, see this [[faq#how_do_i_convert_the_hex_characters_to_ascii|FAQ entry]].

We do not specifically provide support or the details on how to configure your wireless card to connect to the AP.  For linux, this [page](https://web.archive.org/web/20080212235953/http://wirelessdefence.org/Contents/LinuxWirelessCommands.htm) has an excellent writeup.  As well, search the internet for this information regarding linux and Windows systems.  As well, see the documentation for your card's wireless client.  If you are using linux, check the mailing lists and forums specific to the distribution.

Additionally, Aircrack-ng prints out a message indicating the likelihood that the key is correct.  It will look something similar to "Probability: 100%".  Aircrack-ng tests the key against some packets to confirm the key is correct.  Based on these tests, it prints the probability of a correct key.

Also remember we do not support or endorse people accessing networks which do not belong to them.

#### How to convert the hex key back to the passphrase?

People quite often ask if the hexadecimal key found by aircrack-ng can be converted backwords to the original "passphrase".  The simple answer is "NO".

To understand why this is so, lets take a look at how these passphrases are converted into the hexadecimal keys used in WEP.

Some vendors have a wep key generator which "translates" a passphrase into a hexadecimal WEP key.  There are no standards for this.  Very often they just pad short phrases with blanks, zeroes or other characters.  However, usually the passphrases are filled with zeros up to the length of 16 bytes, and afterwards the MD5SUM of this bytestream will be the WEP Key.  Remember, every vendor can do this in a slightly different way, and so they may not be compatible.

So there is no way to know the how long the original passphrase was.  It could as short as one character.  It all depends on the who developed the software.

Knowing all this, if you still wish to try to obtain the original passphrase, Latin SuD has a tool which attempts reverse the process. Click [here](https://www.latinsud.com/wepconv.html) for the tool.

Nonetheless, these passphrases result in a WEP Key that is as easily cracked as every other WEP Key.  The exact conversion method really does not matter in the end.

Keep in mind that wep passwords that look like "plain text" might either be ASCII or PASSPHRASE. Most (all) systems support ASCII and are the default, but some support passphrase and those which support it require users to specify whether it's ascii or a passphrase.  Passphrases can be any arbitrary length.
ASCII are usually limited to 5 or 13 (wep40 and wep104).

As a side note, Windows WZC only supports fixed length hex or ascii keys, so the shortest inputable key is 5 characters long.  See the table above on this page regarding how many characters are needed for specific key lengths.

#### Sample files to try

There are a number of sample files that you can try with aircrack-ng to gain experience:

- [wpa.cap](https://github.com/aircrack-ng/aircrack-ng/raw/master/test/wpa.cap):  This is a sample file with a wpa handshake.  It is located in the "test" directory of the install files.  The passphrase is "biscotte".  Use the password file (password.lst) which is in the same directory.
- [wpa2.eapol.cap](https://github.com/aircrack-ng/aircrack-ng/raw/master/test/wpa2.eapol.cap): This is a sample file with a wpa2 handshake.  It is located in the "test" directory of the install files.  The passphrase is "12345678".  Use the password file (password.lst) which is in the same directory.
- [test.ivs](https://download.aircrack-ng.org/wiki-files/other/test.ivs): This is a 128 bit WEP key file.  The key is "AE:5B:7F:3A:03:D0:AF:9B:F6:8D:A5:E2:C7".
- [ptw.cap](https://github.com/aircrack-ng/aircrack-ng/raw/master/test/wep_64_ptw.cap): This is a 64 bit WEP key file suitable for the PTW method.  The key is "1F:1F:1F:1F:1F".
- [wpa-psk-linksys.cap](https://github.com/aircrack-ng/aircrack-ng/raw/master/test/wpa-psk-linksys.cap): This is a sample file with a WPA1 handshake along with some encrypted packets. Useful for testing with airdecap-ng. The password is "dictionary".
- [wpa2-psk-linksys.cap](https://github.com/aircrack-ng/aircrack-ng/raw/master/test/wpa2-psk-linksys.cap): This is a sample file with a WPA2 handshake along with some encrypted packets. Useful for testing with airdecap-ng. The password is "dictionary".

#### Dictionary Format

Dictionaries used for WPA/WPA bruteforcing need to contain one passphrase per line.

The linux and Windows end of line format is slightly different.  See this [Wikipedia entry](https://en.wikipedia.org/wiki/Line_feed) for details. There are conversion tools are available under both linux and Windows which can convert one format to another.  As well, editors are available under both operating systems which can edit both formats correctly.  It is up to the reader to use an Internet search engine to find the appropriate tools.

However both types should work with the linux or Windows versions of aircrack-ng.  Thus, you really don't need to convert back and forth.

#### Hexadecimal Key Dictionary

Although it is not part of aircrack-ng, it is worth mentioning an interesting piece of work is by SuD.  It is basically a wep hex dictionary already prepared and the program to run it:

``` html
https://www.latinsud.com/pub/wepdict/
```

#### Tools to split capture files

There are times when you want to split capture files into smaller pieces.  For example, files with a large number of IVs can sometimes cause the PTW attack to fail.  In this case, it is worth splitting the file into smaller pieces and retrying the PTW attack.

So here are two tools to split capture files:

- <https://www.badpenguin.co.uk/files/pcap-util>
- <https://www.badpenguin.co.uk/files/pcap-util2>

Another technique is to use Wireshark / tshark. You can mark packets then same them to a separate file.

#### How to extract WPA handshake from large capture files

Sometimes you have a very large capture file and would like to extract the WPA/WPA2 handshake packets from it to a separate file.  The can be done with "tshark" which is a command line version of the Wireshark suite.  Installing the linux version of the [Wireshark suite](https://www.wireshark.org) on your system should also install tshark.

The following command will extract all handshake and beacon packets from your pcap capture file and create a separate file with just those packets:

``` bash
tshark -r <input file name> -R "eapol || wlan.fc.type_subtype == 0x08" -w <output file name>
```

Remember you must use a pcap file as input, not an IVs file.

#### Other Tips

To specify multiple capture files at a time you can either use a wildcard such as * or specify each file individually.

Examples:

- aircrack-ng -w password.lst wpa.cap wpa2.eapol.cap
- aircrack-ng *.ivs
- aircrack-ng something*.ivs

To specify multiple dictionaries at one time, enter them comma separated with no spaces.

Examples:

- aircrack-ng -w password.lst,secondlist.txt wpa2.eapol.cap
- aircrack-ng -w firstlist.txt,secondlist.txt,thirdlist.txt wpa2.eapol.cap

Aircrack-ng comes with a small dictionary called password.lst.  The password.lst file is located in the "test" directory of the source files.  This [[faq#where_can_i_find_good_wordlists|FAQ entry]] has a list of web sites where you can find extensive wordlists (dictionaries).  Also see this [thread](https://forum.aircrack-ng.org/index.php?topic=1373) on the Forum.

Determining the WPA/WPA2 passphrase is totally dependent on finding a dictionary entry which matches the passphrase.  So a quality dictionary is very important.  You can search the Internet for dictionaries to be used.  There are many available.

The [[tutorial|tutorials page]] has the following tutorial [[cracking_wpa|How to crack WPA/WPA2?]] which walks you through the steps in detail.

As you have seen, if there are multiple networks in your files you need to select which one you want to crack.  Instead of manually doing a selection, you can specify which network you want by essid or bssid on the command line.  This is done with the -e or -b parameters.

Another trick is to use John the Ripper to create specific passwords for testing.  Lets say you know the passphrase is the street name plus 3 digits.  Create a custom rule set in JTR and run something like this:

``` bash
john --stdout --wordlist=specialrules.lst --rules | aircrack-ng -e test -a 2 -w - /root/capture/wpa.cap
```

Remember that valid passwords are 8 to 63 characters in length. Here is a handy command to ensure all passwords in a file meet this criteria:

``` bash
awk '{ if ((length($0) > 7) && (length($0) < 64)){ print $0 }}' inputfile
```

or

``` bash
grep -E '^.{8,63}$' < inputfile
```

### Usage Troubleshooting

#### Error message "Please specify a dictionary (option -w)"

This means you have misspelt the file name of the dictionary or it is not in the current directory.  If the dictionary is located in another directory, you must provide the full path to the dictionary.

#### Error message "fopen(dictionary)failed: No such file or directory"

This means you have misspelt the file name of the dictionary or it is not in the current directory.  If the dictionary is located in another directory, you must provide the full path to the dictionary.

#### Negative votes

There will be times when key bytes will have negative values for votes.  As part of the statistical analysis, there are safeguards built in which subtract votes for false positives.  The idea is to cause the results to be more accurate.  When you get a lot of negative votes, something is wrong.  Typically this means you are trying to crack a dynamic key such as WPA/WPA2 or the WEP key changed while you were capturing the data.  Remember, WPA/WPA2 can only be cracked via a dictionary technique.  If the WEP key has changed, you will need to start gathering new data and start over again.

#### "An ESSID is required. Try option -e" message

You have successfully captured a handshake then when you run aircrack-ng, you get similar output:

``` aircrack-ng
Opening wpa.cap
Read 4 packets.
 
         #     BSSID                      ESSID                   ENCRYPTION
         1     00:13:10:F1:15:86                                WPA (1) handshake
Choosing first network as target.
 
An ESSID is required. Try option -e.
```

Solution: You need to specify the real essid, otherwise the key cannot be calculated, as the essid is used as salt when generating the pairwise master key (PMK) out of the pre-shared key (PSK).

So just use -e "<REAL_ESSID>" instead of -e "" and aircrack-ng should find the passphrase.

#### The PTW method does not work

One particularly important constraint is that it only works against arp request/reply packets.  It cannot be used against any other data packets.  So even if your data capture file contains a large number of data packets, if there insufficient arp request/reply packets, it will not work.  Using this technique, 64-bit WEP can be cracked with as few as 20,000 data packets and 128-bit WEP with 40,000 data packets.  As well, it requires the full packet to be captured.  Meaning you cannot use the "-- ivs" option when running airodump-ng.  It also only works for 64 and 128 bit WEP encryption.

#### Error message "read(file header) failed: Success"

If you get the error message - “read(file header) failed: Success” or similar when running aircrack-ng, there is likely an input file with zero (0) bytes. The input file could be a .cap or .ivs file.

This is most likely to happen with wildcard input of many files such as:

``` bash
aircrack-ng -z -b XX:XX:XX:XX:XX:XX *.cap
```

Simply delete the files with zero bytes and run the command again.

#### WPA/WPA2 Handshake Analysis Fails

Capturing WPA/WPA2 handshakes can be very tricky. A capture file may end up containing a subset of packets from various handshake attempts and/or handshakes from more then one client. Currently aircrack-ng can sometimes fail to parse out the handshake properly. What this means is that aircrack-ng will fail to find a handshake in the capture file even though one exists.

If you are sure your capture file contains a valid handshake then use Wireshark or an equivalent piece of software and manually pull out the beacon packet plus a set of handshake packets.

There is an open [GitHub issue](https://github.com/aircrack-ng/aircrack-ng/issues/651) to correct this incorrect behavior.
