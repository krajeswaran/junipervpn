JuniperVPN
==========

Easy, automated python script for logging into VPN from Linux

Credits
=======

Shamelessly lifted from http://makefile.com/.plan/2009/10/juniper-vpn-64-bit-linux-an-unsolved-mystery/.
Also thanks to the excellent `mechanize` python library

Why
===

* Because you can't make JREs work on your system
* Because you want a _faster_ way connect to VPN
* Because you prefer the DIY way(=linux way?)

Notes
=====

There are two issues which are kind of are the reasons for this hack:

* Sun JRE packages are getting removed from repos of major distros. While the replacement - OpenJDK might work in future with Juniper, it doesn't work yet.
* Juniper ships a 32-bit library and expects it to work on 64-bit. In other words, Juniper Networks doesn't really support 64-bit linux. So 64-bit linux needs workarounds OTB.

Installation
============

1. Install the following packages. Some packages might already be in your system, if they are you can safely ignore. Replace `sudo apt-get install` with your distro's package manager. 

    sudo apt-get install build-essential gcc-multilib lib32z1 python-pip 
    sudo pip install mechanize

2. Go to https://rex.corp.ebay.com, select your host and login like you would normally. This will download Juniper's network connect Java applet in `~/.juniper_networks/network_connect/`. Don't worry if you can't connect or the network connect doesn't lanuch. All we need from this step -- is for the files to be downloaed.

3. Now, let's recompile this 32-bit binary to work on your 64-bit box. First cd into,

    cd ~/.juniper_networks/network_connect/
    gcc -m32 -Wl,-rpath,`pwd` -o ncui libncui.so

If you are already on a 32-bit box, just do(haven't tested this, feedback welcome):

    gcc -Wl,-rpath,`pwd` -o ncui libncui.so

4. Get the SSL certificate for your VPN host. Replace `your_host` with something like `webwork-maa.corp.ebay.com`

    echo | openssl s_client -connect your_host:443 2>&1 | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' | openssl x509 -outform der > ssl.crt

5. Download this script and make sure it has the right permissions: `chmod a+x junipervpn.py`

6. Run it for the first time and create the config file - `./junipervpn.py -c`. Your config will be stored in a file called `.junipervpn`. There is a sample config file in this repo, you can use that for a reference.

Now you are ready to go. Run the script, enter the 6-digit code for your secure token and you will be connected. There will be a prompt for password, which is harmless, you can ignore that.

That's it! Enjoy.
