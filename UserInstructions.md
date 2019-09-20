# Clever Yellow Box

A conference wifi network in a box.

### Powering On

* Open the lid and take out the loose cables and packing foam. 
* Do NOT disconnect any other cables
* Plug the ethernet cable into the ethernet wall socket of the conference venue
* Plug the round 3mm jack of the power cable in to the back of the clever yellow box: there is a hole in the outside of the case on the left rear
* Plug the other end into the conference venue power socket and power on
* The clever yellow box will take about 5 minutes to boot up:
  * There is a light under the square logo: it will take about 30 seconds to start flashing
  * The light will flash white while it is booting
  * About 1 minute after it stops flashing and turns steady blue, the wifi network should be available
* The wifi password should be on a separate piece of paper in the box, or will be given to you some other way
* Do NOT share the wifi password with conference venue staff or write it in a place they can see it, or announce it loudly from the front. Hand it out discreetly to attendees: on a separate piece of paper each. Your security depends on this.
* During normal operation, leave the lid open and make sure nothing covers the box. Some of the components get quite hot and need air to circulate to keep them cool.

### File Sharing

You can use the clever yellow box to share files with each other or with the person doing data projection.

On a Mac
* Open Finder
* In the Go menu, choose "Connect to Server"
* A box will open: type **smb://cleveryellowbox/cleveryellowbox** and click Connect
* Choose Guest mode if you are asked to log in

On Windows 10
* Open File Explorer
* Right click on This PC
* Click "Add a network location"
* Type **//cleveryellowbox/cleveryellowbox**
* Follow the other instructions in the wizard. If asked for a username and password, leave them blank or choose Guest mode

see https://www.techrepublic.com/article/how-to-connect-to-linux-samba-shares-from-windows-10/ for
 
### Using a 3G/4G/Cellular modem

If the venue does not provide an ethernet jack you can plug in to, you may be able to use a third party 3G hotspot device.

* Power the device on.
* Log into it following the manufacturers instructions
* Turn off any WiFi hotspot (if it has one): it may interfere with the clever yellow box
* Change any other settings that are needed to get internet working
* Connect the clever black box to the ethernet port of your device
* Wait a few minutes for the software on both devices to automatically configure
* Use as normal. Advise conference attendees that data is limited and not to watch youtube or install large windows updates.

### Packing up / After the conference

* If you have used the file sharing, log in and delete all the files so the box is clean for the next user
* Power off. Disconnect cables from the venue and unplug the power cable from the rear of the clever yellow box
* Place the packing foam over the internal components
* Coil the cables and put them on top of the foam
* Clip the lid closed. Take care not to jam the cables in the hinge / edge

### Maintenance

Automatic updates are switched on for most things. One or two need a human to do it.
Once every 6 months, someone needs to log in remotely and update the operating system and controller software.

### Help

If the clever yellow box can connect to the internet, there is a remote configuration tool that a technical helper can access via UniFi cloud to help troubleshoot.

If it cannot connect to the internet, you can still use it for file sharing

Do NOT power it off and on again quickly to try and "fix" it. You will kill the solid state hard drives inside. If you need to power off, wait at least 15 seconds before powering on again. Wait until the blue light is steady (not flashing) before powering off.