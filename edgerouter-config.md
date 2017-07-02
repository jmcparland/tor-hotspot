# VLAN DEFINITION:

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

# PORT FORWARDING:

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
