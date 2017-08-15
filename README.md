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
mkdir /home/pi/vpnconfig
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

Once you're connected to your Pi's wifi, you should be able to access the web interface using the following URL:

http://192.168.55.1:5000


Customizing the Web Interface
=============================

In general, you should be done at this point. The web interface allows you to query information about your internet and VPN connections, and you can connect to a wifi network. The web interface is configured not to offer your Pi's own wifi network for connection. This is done in ~/wiconfig/app/templates/index.html in line 64:

```
{% if network.ssid != "PBox" %}
```

I named my Pi's wifi network "PBox". If you're using a different name just change this line.

The web interface also contains a possibility to change the VPN configuration. Many VPN companies provide configuration files that give you an IP address in your country of choice. So it might be a good idea to have some of these files on your Pi to be able to select the suitable one on the fly.

Copy these files (typically \*.ovpn files) to the directory /home/pi/vpnconfig or a subfolder to this directory. The web interface will offer these files for selection, and it will reflect the directory structure so it's easier for you to find the right file. (E.g., you might have one subfolder for each target country.)

When selected, the web interface service will copy the selected file to /etc/openvpn/client.conf. In case you need credentials for your connections, these credentials need to reside in the /etc/openvpn/ folder, and your \*.ovpn file needs to point to the file containing the credentials, such as

```
auth-user-pass /etc/openvpn/pwd.txt
```

Recommended stuff
=================

I recommend installing the **haveged** package on the Raspberry Pi. This will speed up all the encryption stuff tremendeously.

```
sudo apt-get install haveged
```

In case you're using Android devices with your mobile VPN router, you might find the [Hot Button SSH Command Widget](https://play.google.com/store/apps/details?id=crosien.HotButton) useful. I use it to shutdown or reboot my Pi from the mobile device, without having to open the web interface first. In the typical use case, I will be connected to my Pi's wifi anyway, so the widget can send the shutdown or reboot command by SSH as well.
