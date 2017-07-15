Screenshots of relevant Ubiquiti Controller configuration pages:
* **01 Ubiquiti Controller Wireless Network Settings**
  * Defines the SSID name to be broadcast
  * Instructs to use Guest Policies
  * Defines the VLAN to use.
* **02 Ubiquiti Controller Guest Policy Settings**
  * No authentication required.
  * Redirects to my "promotional page" describing the project.
  * Pre-Authorization Access gives permission to see the Ibiquiti controller (192.168.4.7 in my case) and the local Tor LAN (including the landing page / router 172.29.3.1).
* **03 Ubiquiti Controller Guest User Group**
  * Defines a user group with bandwidth throttling.

That's it for the wireless!
