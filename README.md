# tor-hotspot
Add a Tor hotspot to an established network using Ubiquiti gear (EdgeRouter-X &amp; Unifi AP AC Lite) by adding a Raspberry Pi. Generalizes to VLAN-capable network gear.

This is a high-level description of an operational network environment providing an add-on Tor hotspot service to the environment; it is not a howto. The descriptioni assumes some fluency with networking fundamentals (routing, VLANs, DHCP, firewalling, etc.), fluency with linux use and configuration (a Raspberry Pi running raspian hosts the Tor service), and assumes network equipment capable of creating a separate wireless network for the service (I use inexpensive Ubiquiti gear, which I will describe). Lacking any of the above, this is a good, fairly inexpensive project to put one on the path to acquiring those items while providing an unusual service for your and your neighbors.

ABOUT TOR:

Tor is a mature, cooperative, and sometimes controversial network that provides free, network-anonymized internet access for users. See the Tor Project website for additional information:

https://www.torproject.org/

OBJECTIVES:

(1) Add an open wireless network to an existing network that automatically routes all browsing through the public Tor network.
(2) The wireless network should serve at least as a Tor Relay Node (a) to assist the public Tor network and (b) to obfuscate Tor network traffic originating here.
(3) The added service should not overly impact overall network performance and should not threaten the security of the primary network.

APPROACH / STRATEGY:

(1) Using inexpensive but capable commercial network gear, create a VLAN to contain the Tor service's traffic with its sole interface on the firewall-router.
(2) Add a Raspberry Pi (or similarly capable device, VM, etc.) to the VLAN with a static IP address and default route to the firewalll-router.
(3) Configure DHCP on the VLAN to assign the Raspberry Pi as the default gateway for all computers joining the network through the VLAN.
(4) Install and configure Tor services on the Raspberry Pi. Configure firewall rules allowing inbound forwarding of Tor traffic to the Pi.
(5) Add an SSID identifying the wireless access to the VLAN-capable wireless system which directs all wireless traffic on that SSID to the new VLAN.
(6) Configure bandwidth rate limiting one or both of the wireless access point and Raspberry Pi.
(7) Configure firewall rules to isolate the new VLAN from other network segments, and rules to ensure that ONLY the Pi can communicate through the frewall-router from inside the VLAN -- no other wireless client traffic should exit.

EQUIPMENT LIST:

* Ubiquiti EdgeRouter-X (~$50)
* Ubiquiti Unifi AP AC Lite (~$80)
* Raspbery Pi (~$50)
