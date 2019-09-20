# Clever Yellow Box

A conference wifi network in a box.

This repository contains
- What it does / why we need it
- Instructions to build and run your own
- Supplier and cost data

Specifications
- Small / portable / rugged unit to cope with airline travel
- Easy turnkey operation for end users (plug the internet *here* and the power *there*)
- Secure WiFi network for 50-200 users
- Firewall to protect from potentially hostile equipment on venue networks 
- Client isolation to protect users from other users with possibly infected devices
- File sharing to exchange conference slides 

## History

I work for an organisation that has conferences around Asia and the Arab world. 
We know not to trust the WiFi networks at airports, hotels, conference venues for many reasons.

We looked at some consumer grade devices (like a FritzBox) which have many of these functions but do not
* handle double-NATing well
* handle 50+ wifi devices
* have regular enough software updates
* offer the level of secure remote access needed 

In 2017 we asked a friend to build us a small unit with the above specifications. 
It came in a bright yellow instrument case, so we named it the **Clever Yellow Box**. That name has stuck with us.
It also had optional VPN backhaul, and a 3G modem in case the venue had no ethernet. 
After returning the prototype following a successful conference, we were unable to contact him again. Which was a pity. 
We wanted to make more! Learning from his design, I bought some equipment and started to tinker. 
We found a local company to do the technical bits but the owner had a series of illnesses and was unable to complete the project.
So the material below is just me tinkering and trying to get it to work. Advice and pull requests are welcome. 
Specifically, we'd like your help with
* VPN backhaul (a second WiFi network that tunnels out to a safe location)
* 3G/4G connection: a _simple_ cellular modem that takes a local SIM card and an ethernet cable. PoE is available.

## Components

All prices in New Zealand Dollars including 15% GST

| Component | Price | Supplier | Function |
| :--- | ---: | :--- | :--- |
| [UniFi USG 3P](https://www.ui.com/unifi-routing/usg/) | $175.56 | GoWifi | Firewall |
| [UniFi UAP-AC_LITE](https://www.ui.com/unifi/unifi-ap-ac-lite/) | $130.20 | GoWifi | WiFi access point |
| [Raspberry Pi Model 3B+](https://www.raspberrypi.org/products/raspberry-pi-3-model-b-plus/) | $64.84 | PBTech | UniFi Controller, Samba Server, other tools if needed |
| Apacer 16GB MicroSDHC card | $9.32 | PBTech | Hard disk for Raspberry Pi. Slightly larger than needed to allow room for Samba file sharing |
| Raspberry Pi Case | $10.61 | PBTech | Mounting point for Raspberry Pi |
| [5 Port PoE Switch](https://www.jaycar.co.nz/5-port-10-100-poe-network-switch/p/YN8074) | $124.90 | Jaycar | PoE injector for all the things. | 
| [12V PoE Splitter](https://www.jaycar.co.nz/poe-power-splitter/p/YN8414) | $29.90 | Jaycar | Power for USG |
| [MicroUSB PoE Splitter](https://www.jaycar.co.nz/5v-micro-usb-poe-splitter/p/YN8416) | $14.90 | Jaycar | Power for Raspberry Pi. Could have used a [PoE hat](https://www.raspberrypi.org/products/poe-hat/) but these are $45+ | 
| 2.5mm DC power [connector](https://www.jaycar.co.nz/2-5mm-bulkhead-male-dc-power-connector/p/PS0524) | $5.30 | Jaycar | Mount a power socket on top of the case |
| Cat6 [Socket](https://corysadvantage.co.nz/products/product/0000934616) and [Module](https://corysadvantage.co.nz/products/product/0000936441) | $22.00 | JM Russell | Mount an ethernet jack for top of case |
| [ABS MPV2 Case](https://www.jaycar.co.nz/abs-instrument-case-with-purge-valve-mpv2/p/HB6381) | $80.90 | Jaycar | Travel case |
| Alu Plate x 2 | $28.80 | Jaycar | Mount components inside case |
| M3 Nuts & Bolts | $6.20 | Jaycar | Mount components inside case | 
| **Total** | **$703.23** | | so far... |
   
Plus Cat 6 ethernet cable and connectors, electrical tape etc

I chose UniFi because
* the price is good
* their cloud management tools are simple to use: which means we can maintain the system remotely
* they have regular software updates, which is essential in the current security environment
* individual components are swappable. So if you want more WiFi users or wider coverage, buy a bigger access point  
* others in the community know and love UniFi gear: so we have support if needed

We chose the PoE switch because it was small / light. It only does 10/100 (not Gigabit ethernet) but we hope this 
will not be a problem because the kinds of venues we go to do not have gigabit internet connections anyway. 
We are lucky to get ADSL speeds (2-8Mbps) or may need cellular backhaul.

Thanks to [Beta Solutions](https://www.betasolutions.co.nz) for their help locating components.


## Setup Instructions

### Raspberry Pi

* Download Raspbian Stretch Lite from https://www.raspberrypi.org/downloads/raspbian/
* Use [Etcher](https://www.balena.io/etcher/) to put the image on the SD card
* Put an empty file called "ssh" in the root of the SD card (`touch ssh`) to enable headless setup
* Eject the card and install it in the Raspberry Pi
* Power up the Pi and connect it to the local network. Wait a minute for it to boot.
* Find your Pi IP Address eg 192.168.1.11
* `ssh pi@192.168.1.11` password is `raspberry`. [More Help](https://hackernoon.com/raspberry-pi-headless-install-462ccabd75d0)
* `passwd` to change the default password to something more secure (Do This First!)
* `mkdir /home/pi/.ssh`
* `chmod og-rwx .ssh`
* copy your ssh public key to `/home/pi/.ssh/authorized_keys`
* `chmod og-rwx .ssh/authorized_keys`
* exit and log in again with no password. You will be rebooting / logging in a few times so this step is worth it.
* `sudo apt update` then `sudo apt upgrade` to upgrade to the latest OS
* `sudo apt install unattended-upgrades` to keep the OS up to date
* `sudo rpi-update` to update the firmware to the latest 
* `sudo reboot` 
* `sudo raspi-config` 
  * 2 Network. Set Network name eg `cleveryellowbox`
  * 2 Network. Enable predictable network interface names: Yes
  * Advanced Options: Expand File System
* `sudo timedatectl set-timezone UTC` to select UTC timezone  
* edit `/boot/config.txt` and add the following 2 lines to disable wifi and bluetooth (see https://raspberrypi.stackexchange.com/questions/43720/disable-wifi-wlan0-on-pi-3)
```
dtoverlay=pi3-disable-wifi
dtoverlay=pi3-disable-bt
```
* comment out `dtparam=audio=on` if you wish

### Samba

Optional: set up the raspberry pi as a tiny file server to share files with other conference attendees eg to send slides to the projectionist
* `sudo apt install samba`
* `sudo mkdir -m 1777 /share`
* `sudo nano /etc/samba/smb.conf`. Add the following to the end of the file
```
[cleveryellowbox]
Comment = Clever Yellow Box
Path = /share
Browseable = yes
Writeable = Yes
only guest = yes
create mask = 0777
directory mask = 0777
Public = yes
Guest ok = yes
guest account = ftp
```
* `sudo service smbd restart`

### UniFi Controller Install

* install the UniFi controller on the raspberry pi: follow the instructions https://help.ubnt.com/hc/en-us/articles/220066768-UniFi-How-to-Install-Update-via-APT-on-Debian-or-Ubuntu. Method A works best for installing the keys.
* reboot (because sometimes the controller doesn't start after install). It takes several minutes for the unifi controller to boot.
* visit https://cleveryellowbox:8443 or https://192.168.1.7:8443 to finish the setup
* from the setup wizard:
  * enable auto backup. 
  * Set time zone to UTC. 
  * Connect to ubnt cloud account so you can access it remotely 
* if this doesn't work, visit `/var/log/unifi` to look for error messages in the log files, or try `service unifi status` which may hint at the problem
  * people frequently report problems with the wrong java version installed by default or with mongodb 
  * try `apt install openjdk-8-jre-headless` _first_ before installing unifi 
  * java certificates sometimes get messed up if you install a newer jre. Try `update-ca-certificates` 

### Physical Connections

Disconnect the Raspberry Pi from the local network and connect it to the PoE Switch along with all the other components
* USG: WAN1 to the internet
* USG: LAN1 to the Link port of the switch
* Raspberry Pi: to any other switch port
* AP: to any other switch port
* Your laptop: to a spare switch port so you can access the controller

### UniFi Controller Configuration (via the web interface)

Controller is usually at https://192.168.1.7:8443. Browse to that address in a web browser. Log in
* Go to Devices
* You should see 2 devices: the USG and the AP
* Apply any updates _before_ adopting. Updates may take > 10 minutes per device, may require a power cycle etc.
  * beware: the USG defaults to serve DHCP from 192.168.1.1. So it will fail to upgrade or adopt if attached to a 192.168.1.0/24 network 
* Devices tab: Adopt each unifi device and set an alias for them
* Clients tab: There should be two clients: your laptop and the raspberry pi. Set aliases for them
  * For the raspberry pi, go to Settings, Use Fixed IP and type the current IP address so it will always get the same IP
* Settings | Site 
  * check the timezone is UTC
  * set an appropriate country, which defines the wifi channels allowed
  * switch on automatic upgrades. This will upgrade the devices (USG and AP) but _not_ the controller
* Settings | Guest Control
  * Pre-authorization access: add 192.168.1.7 (the fixed IP of the raspberry pi) which allows your guests to see the samba server and allows you (as guest) to see the controller  
* Settings | Wireless Networks | Create New Wireless Network
  * Name: `cleveryellowbox`
  * WPA Personal
  * Sensible password
  * Apply Guest Policies
* Settings | Services | MDNS
  * Enable Multicast DNS so that guest users can see the name `cleveryellowbox`
* Settings | Controller 
  * set controller name to `cleveryellowbox`
  * set controller host name to `cleveryellowbox`
* Settings | Cloud Access
  * Log in 
  * it can take quite a few minutes before the controller is accessible from your cloud account   
  
You probably want to change the default IP address / DHCP range to avoid collisions with conference venue networks.
Choose a random IP range eg 192.168.123.1 (where the 123 is something you think will be unique / unlikely at a venue).
* Settings | Networks | LAN (Edit)
  * Change Gateway / Subnet to your new IP address
  * Update the DHCP range to match eg 192.168.123.6 - 192.168.123.254
This will break lots of things: your controller will not be able to adopt the USG and AP. I'm not quite sure how I did 
it but after multiple reboots, factory resets, forced adoptions etc i finally got the controller to adopt the USG and AP
at their new addresses.
* Set a "static" IP for the controller. 
  * In Clients, click on the raspberrypi controller. 
  * In Confuguration | Network, set the IP address to its current IP: probably 192.168.123.7
*     
  
### Physical Build

Cut the aluminium plate to 213x295mm and file in some grooves to allow it to slide to the bottom past the stays.
I drilled a hole in the back to put the power cable in, and small screw holes in the lid for the wifi AP, 
and through the base to hold the plate down and stop it moving in transit.  

Ethernet cables were custom made to length.

![Detail](/detail.jpeg)
![Finished](/finished.jpeg)

### Maintenance

Automatic updates are switched on for most things. The unifi controller software and raspbian OS may need manual hands on updates once every 6 months.
  
## To Do
 
* include photos of final setup
* include logical wiring diagram 
 
Look at cellular modems
* Robustel PoE with lots of bands incl China. Antennae and power separate https://www.pbtech.co.nz/product/NETRTL2001/Robustel-R2000-4L-2G3G4G-LTE-Router-Dual-SIM-2xEth
* Smaller cheaper modem https://www.pbtech.co.nz/product/NETDLK4921/D-Link-DWR-921-C3-4G-LTE-Router-Wireless-N300300Mb but ancient firmware and no manufacturer updates
* wAP LTE kit from MikroTik https://mikrotik.com/product/wap_lte_kit#fndtn-specifications (Small, PoE RouterOS gets regular updates https://mikrotik.com/download/changelogs)
* https://www.digi.com/products/networking/cellular-routers/digi-transport-wr31#partnumbers 
* https://www.pbtech.co.nz/product/NETDTK0161/DrayTek-DV2862LAC-Triple-WAN-Router-LTE-ADSLVDSL-U  

Cases:
* Pelican US store https://www.pelican.com/us/en/products/cases, NZ supplier https://www.rubbermonkey.co.nz/Cases-Bags/Pelican-Hard-Cases

Samba
* probably should partition the raspberry pi drive so that the samba share does not gobble up storage needed for the OS 
* get some advice about filesystem permissions

UniFi Controller
* figure out how to automatically upgrade the controller: avoiding the backup prompt that requires human interaction

