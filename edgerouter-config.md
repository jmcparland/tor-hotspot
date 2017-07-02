This document generally defines and walks through relevant configuration details for the Ubiquiti EdgeRouter-X. Separate documents will be provided for configuration of the wireless access and the Raspberry Pi Tor server ("stormbeam," in our example). Note that this document details a particular implementation and may need to be adjusted significantly to fit into your network. Particular IP & MAC addresses may be obfuscated below.

Covered here:
* VLAN Definition
* DHCP Service Configuration
* Port-Forwarding for Inbound Tor Relay Traffic
* Firewall Rules, Inbound & Outbound

# VLAN DEFINITION

In my situation, EdgeRouter port eth3 is used for the underlying LAN and has several VLANs defined above it as "virtual interfaces (vif)." In this case, VLAN 30 is defined for the "Neighborhood" network, with gateway interface address 172.29.3.254. There are referneces to firewall rulesets for both inbound and outbound traffic (shown later).

```
ubnt@ubnt# show interfaces ethernet eth3 vif 30                                                                                                        
 address 172.29.3.254/24                                                                                                                               
 description Neighborhood                                                                                                                              
 firewall {                                                                                                                                            
     in {                                                                                                                                              
         name NGH_IN                                                                                                                                   
     }                                                                                                                                                 
     out {                                                                                                                                             
         name NGH_OUT                                                                                                                                  
     }                                                                                                                                                 
 }                                                                                                                                                     
[edit]
ubnt@ubnt#  
```

# DHCP SERVICES

DHCP services for the network are defined for the VLAN on the EdgeRouter. As a client joins the network, it is assigned an IP address in the 172.29.3.25-45 block, told to use Google (8.8.8.8) or OpenDNS (4.2.2.2) for name services, and most importantly you can see it is told to use 172.29.3.1 as a deault gateway. Just below, you can see that a particular computer named "stormbeam" is assigned this address when it comes online; this is the Raspberry Pi. [The MAC address of this particular device is obfuscated below.]

```
ubnt@ubnt# show service dhcp-server shared-network-name Neighbors                                                                                      
 authoritative disable                                                                                                                                 
 subnet 172.29.3.0/24 {                                                                                                                                
     default-router 172.29.3.1                                                                                                                         
     dns-server 8.8.8.8                                                                                                                                
     dns-server 4.2.2.2                                                                                                                                
     lease 1800                                                                                                                                        
     start 172.29.3.25 {                                                                                                                               
         stop 172.29.3.45                                                                                                                              
     }                                                                                                                                                 
     static-mapping stormbeam {                                                                                                                        
         ip-address 172.29.3.1                                                                                                                         
         mac-address b8:27:eb:XX:XX:XX                                                                                                                
     }                                                                                                                                                 
 }                                                                                                                                                     
[edit]
ubnt@ubnt#                                                                                                                                             
```

# PORT FORWARDING

As part of our operation as a public Tor Relay Node, we need to accept inbound Tor traffic for relay. The traditional port for Tor traffic is 9090. In our case, we have configured stormbeam to *advertise* using port 443 and to *accept* traffic on port 9090. This port-forwarding rule takes inbound port 443 traffic and fortwards it through the firewall to stormbeam on port 9090 for handling:

```
ubnt@ubnt# show port-forward rule 2                                                                                                                    
 description "tor relay to stormbeam"                                                                                                                  
 forward-to {                                                                                                                                          
     address 172.29.3.1                                                                                                                                
     port 9090                                                                                                                                         
 }                                                                                                                                                     
 original-port 443                                                                                                                                     
 protocol tcp                                                                                                                                          
[edit]
ubnt@ubnt#  
```
# Firewall Rules

As a reminder, "inbound" rules apply to traffic arriving at the firewall-router _from_ the Tor VLAN, while "outbound" rules apply to traffic at the firewall-router destined to the Tor VLAN.

## Inbound

Disclaimer: These rules are functional, but not guaranteed to be "best practice." Addtionally, they do contain specific addressses on this network's LAN which may be changed or ommitted for your individual use cases.

The gist of the rules below:

* Rules 10 & 20: On-going (previously established) connections can continue, but discard invalid packets.
* Rule 30: Anything from server stormbeam is allowed out to the network or to the internet.
* Rule 40: Any guest on the VLAN can talk to stormbeam. (This may not actually be necessary since all devices in the VLAN are in the same broadcast domain.)
* Rule 50: In this implementation, wireless clients go through a captive portal and are redirected to an "About Tor" webpage on the VLAN. This rule allows the wireless client to speak to the wireless controller (outside of the VLAN) as part of that authentication process.
* Rule 60: Server stormbeam was already allowed to talk to anyone. This rule ensures that no other client on the VLAN can talk to any other device on the larger network outside of the VLAN (except for the exception in Rule 50). [Wireless policy rules will also prevent wireless clients on the VLAN from talking to one another.]
* Rule 70: Explicitly disables outbound UDP traffic, to include network-bound DNS lookups.
* Rule 80: Explicitly disables outbound ICMP traffic (e.g. "ping") from clients, which would bypass the tor network.

All traffic not explicitly handled by the above rules is accepted.

Remember, all guest wireless clients to the VLAN are instructed to use stormbeam as a gateway.

```
ubnt@ubnt# show firewall name NGH_IN                                                                                                                   
 default-action accept                                                                                                                                 
 description "allow all internet-bound traffic"                                                                                                        
 enable-default-log                                                                                                                                    
 rule 10 {                                                                                                                                             
     action accept                                                                                                                                     
     description "allow related,established"                                                                                                           
     log disable                                                                                                                                       
     protocol all                                                                                                                                      
     state {                                                                                                                                           
         established enable                                                                                                                            
         invalid disable                                                                                                                               
         new disable                                                                                                                                   
         related enable                                                                                                                                
     }                                                                                                                                                 
 }                                                                                                                                                     
 rule 20 {                                                                                                                                             
     action drop                                                                                                                                       
     description "drop invalid"                                                                                                                        
     log disable                                                                                                                                       
     protocol all                                                                                                                                      
     state {                                                                                                                                           
         established disable                                                                                                                           
         invalid enable                                                                                                                                
         new disable
         related disable                                                                                                                               
     }                                                                                                                                                 
 }                                                                                                                                                     
 rule 30 {                                                                                                                                             
     action accept                                                                                                                                     
     description "allow stormbeam"                                                                                                                     
     log disable                                                                                                                                       
     protocol all                                                                                                                                      
     source {                                                                                                                                          
         address 172.29.3.1                                                                                                                            
     }                                                                                                                                                 
 }                                                                                                                                                     
 rule 40 {                                                                                                                                             
     action accept                                                                                                                                     
     description "allow guest to stormbeam"                                                                                                            
     destination {                                                                                                                                     
         address 172.29.3.1                                                                                                                            
     }                                                                                                                                                 
     log disable                                                                                                                                       
     protocol all                                                                                                                                      
 }                                                                                                                                                     
 rule 50 {                                                                                                                                             
     action accept
     description "allow guest wireless authentication"                                                                                                 
     destination {                                                                                                                                     
         address 192.168.xx.yy                                                                                                                          
         port 8880                                                                                                                                     
     }                                                                                                                                                 
     log disable                                                                                                                                       
     protocol tcp                                                                                                                                      
 }                                                                                                                                                     
 rule 60 {                                                                                                                                             
     action reject                                                                                                                                     
     description "reject guest to priv addrs"                                                                                                          
     destination {                                                                                                                                     
         group {                                                                                                                                       
             network-group private_addrs                                                                                                               
         }                                                                                                                                             
     }                                                                                                                                                 
     log disable                                                                                                                                       
     protocol all                                                                                                                                      
 }                                                                                                                                                     
 rule 70 {                                                                                                                                             
     action reject                                                                                                                                     
     description "reject other guest udp"                                                                                                              
     log disable
     protocol udp                                                                                                                                      
     source {                                                                                                                                          
     }                                                                                                                                                 
 }                                                                                                                                                     
 rule 80 {                                                                                                                                             
     action reject                                                                                                                                     
     description "reject icmp out"                                                                                                                     
     log disable                                                                                                                                       
     protocol icmp                                                                                                                                     
 }                                                                                                                                                     
[edit]
ubnt@ubnt#                                                                                                                                             
```

## Outbound

Disclaimer: See the "Inbound" Rules disclaimer above.

These rules handle traffic headed into the VLAN / toward stormbeam and the wireless clients. Gists of the following rules:

* Rule 10: Already established connections may continue.
* Rule 20: Invalid packets are dropped.
* Rule 30: _This is a non-essential rule._ This system has Nagios monitoring the stormbeam server to make sure it's accessible and alive. The Nagios service does not reside inside the Tor VLAN, so this rule grants access.
* Rule 40: _This is a non-essential rule_ that permits one particular administrative computer with a defined MAC address permission to access the Tor VLAN from outside the VLAN for systems checks, maintenance, etc.
* Rule 50: _This is a non-essential rule_ that allows an entirely different office subnet into the Tor VLAN to use the Tor services via proxy.
* Rule 51: _This is a non-essential rule_ that allows systems on a particular administrative subnet access to the Tor VLAN.

Note that:
1. Traffic not explicitly allowed is dropped, except...
2. The port-forwarding rules previously described supercede defined firewall rules. That is, port-forwarding on Ubiquiti systems separately "punches holes" through the firewall with implicit rules.

```
ubnt@ubnt# show firewall name NGH_OUT                                                                                                                  
 default-action drop                                                                                                                                   
 description ""                                                                                                                                        
 rule 10 {                                                                                                                                             
     action accept                                                                                                                                     
     description "allow related,established"                                                                                                           
     log disable                                                                                                                                       
     protocol all                                                                                                                                      
     state {                                                                                                                                           
         established enable                                                                                                                            
         invalid disable                                                                                                                               
         new disable                                                                                                                                   
         related enable                                                                                                                                
     }                                                                                                                                                 
 }                                                                                                                                                     
 rule 20 {                                                                                                                                             
     action drop                                                                                                                                       
     description "drop invalid"                                                                                                                        
     log disable                                                                                                                                       
     protocol all                                                                                                                                      
     state {                                                                                                                                           
         established disable                                                                                                                           
         invalid enable                                                                                                                                
         new disable                                                                                                                                   
         related disable
     }                                                                                                                                                 
 }                                                                                                                                                     
 rule 30 {                                                                                                                                             
     action accept                                                                                                                                     
     description "allow nagios"                                                                                                                        
     log disable                                                                                                                                       
     protocol icmp                                                                                                                                     
     source {                                                                                                                                          
         address 192.168.xx.yy                                                                                                                          
     }                                                                                                                                                 
 }                                                                                                                                                     
 rule 40 {                                                                                                                                             
     action accept                                                                                                                                     
     description "allow admin computer"                                                                                                            
     log disable                                                                                                                                       
     protocol all                                                                                                                                      
     source {                                                                                                                                          
         mac-address 11:22:33:44:55:66                                                                                                                 
     }                                                                                                                                                 
 }                                                                                                                                                     
 rule 50 {                                                                                                                                             
     action accept                                                                                                                                     
     description "allow home TMP"
     log disable                                                                                                                                       
     protocol all                                                                                                                                      
     source {                                                                                                                                          
         address 192.168.zz.xx/24                                                                                                                        
     }                                                                                                                                                 
 }                                                                                                                                                     
 rule 51 {                                                                                                                                             
     action accept                                                                                                                                     
     description "allow admin TMP"                                                                                                                     
     log disable                                                                                                                                       
     protocol all                                                                                                                                      
     source {                                                                                                                                          
         address 192.168.mm.nn/24                                                                                                                        
     }                                                                                                                                                 
 }                                                                                                                                                     
[edit]
ubnt@ubnt#                                                                                                                                             
```
