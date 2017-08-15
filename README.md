WLAN-VPN-Pi
===========

This repository contains scripts and code to use a Raspberry Pi as a Wi-Fi access point (wlan0) that connects to the Internet via the following interfaces:

* Ethernet (eth0)
* An additional WLAN interface (wlan1)

Connection to the Internet is NATed and a VPN client tunnel to a VPN server is used to encrypt ALL traffic to and from ALL devices connected to the access point. Alternatively, direct NATing without a VPN tunnel is also possible.

OpenVPN in client mode is used for creating the tunnel to an OpenVPN server somewhere on the Internet. A great project description of how to set-up a VPN server on the other end at home (on a Raspberry Pi, of course) can be found at http://readwrite.com/2014/04/10/raspberry-pi-vpn-tutorial-server-secure-web-browsing. Alternatively it's also possible to use a commercial service that offers OpenVPN server connectivity.

Hardware Requirements: 
Option 1: Raspberry Pi 3 with built-in Wi-Fi. Backhaul is possible via Ethernet of Wi-Fi. If Wi-Fi is used a USB Wi-Fi dongle is required (see below).

Option 2: Raspberry Pi 1/2 (without built-in Wi-Fi). Apart from a Raspberry Pi, a USB Wi-Fi dongle is necessary. If Wi-Fi is used as a backhaul, a second Wi-Fi USB dongle is required. This project uses a hostapd executable that was compiled for the Realtek RTL8188CUS chipset. The following devices have been checked for compatibility:

* EDIMAX EW-7811UN

For other chipsets, other hostapd executables might be required. 

NOTE THAT THE USE OF **THIS** OPTION IS STRONGLY DISCOURAGED AS CURRENT KERNELS BREAK THE HOSTAPD FUNCTIONALITY. AS A CONSEQUENCE KERNEL UPDATES ARE BLOCKED. USE A RASPBERRY PI 3 FOR THIS PORJECT WHICH DOES NOT SUFFER FROM THIS PROBLEM.

Have a look at the project's wiki for installation and use instructions: 

https://github.com/martinsauter/WLAN-VPN-Pi/wiki

Setting up the web interface
============================

After setting up your Raspberry Pi as shown in Matin Sauter's excellent tutorial, it's time to enhance your mobile VPN router with a fancy web configuration interface.

I adopted an idea by Daniel S. Pierson and extended it to fit my needs. The original paper can be downloaded at http://digitalcommons.calpoly.edu/cscsp/83/.

First of all, mae sure you have followed Martin Sauter's instructions to set up the whole VPN stuff. Then, follow these steps:

Log on to your Raspberry Pi and install Python and Virtualenv (I assume you're doing all this as user "pi"):

```
sudo apt-get install python-dev python-virtualenv
```

Next, we set up the Flask web server inside a Python virtual environment plus the necessary packages:

```
cd /home/pi
virtualenv wiconfig
. wiconfig/bin/activate
pip install Flask
pip install netifaces
pip install wifi
```

Now download the file wiconfig.tar.gz to pi's home directory and extract its contents:

```
tar -xzf wiconfig.tar.gz /home/pi
```

This file contains all of the web interface files, i.e. the HTML template, images and scripts, CSS files and the Python script that does all the magic.

Finally, we need to activate the web server at system startup.

```
sudo nano /etc/rc.local
```

Add the following lines to the end of the file, right before the line that reads "exit 0":

```
# start webinterface
cd /home/pi
. wiconfig/bin/activate
python wiconfig/run.py &
```

Save the file (Ctrl+O), close nano (Ctrl+X) and reboot your Pi:

```
sudo reboot
```
