JuniperVPN
==========

Easy, automated python script for logging into Juniper VPN(Network Connect) from Linux

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

Todo
====

* If host changes in config, auto-fetch ssl certificate for that host

Installation
============

1. Install the following packages. Some packages might already be in your system, if they are you can safely ignore. Replace `sudo apt-get install` with your distro's package manager. 

    <pre><code>
    sudo apt-get install build-essential gcc-multilib lib32z1 python-pip
    sudo pip install mechanize
    </code></pre>

2. Go to https://rex.corp.ebay.com, select your host and login like you would normally. This will download Juniper's network connect Java applet in `~/.juniper_networks/network_connect/`. Don't worry if you can't connect or the network connect doesn't lanuch. All we need from this step -- is for the files to be downloaed.

3. Now, let's recompile this 32-bit binary to work on your 64-bit box. If you are already on a 32-bit box, just do(haven't tested this, feedback welcome) - ``gcc -Wl,-rpath,`pwd` -o ncui libncui.so``


    <pre><code>
    cd ~/.juniper_networks/network_connect/
    gcc -m32 -Wl,-rpath,`pwd` -o ncui libncui.so
    </code></pre>

4. Get the SSL certificate for your VPN host. Replace `your_host` with something like `webwork-maa.corp.ebay.com`

    <pre><code>
    echo | openssl s_client -connect your_host:443 2>&1 | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' | openssl x509 -outform der > ssl.crt
    </code></pre>

5. Download this script and make sure it has the right permissions: `chmod a+x junipervpn.py`

6. Run it for the first time and create the config file - `./junipervpn.py -c`. Your config will be stored in a file called `.junipervpn`. There is a sample config file in this repo, you can use that for a reference.

Now you are ready to go. Run the script, enter the 6-digit code for your secure token and you will be connected. There will be a prompt for password, which is harmless, you can ignore that.

That's it! Enjoy.

Ubuntu 12.04 issue
------------------

The problem is Juniper's VPN(ncsvc to be specific) tries to change `/etc/resolv.conf` and fails because it can't find the file. Ubuntu 12.04 LTS changes the way `resolv.conf` file works -- see http://www.stgraber.org/2012/02/24/dns-in-ubuntu-12-04/. This apparently is done by package `resolvconf`. 

Way to keep things predictable on a LTS release! 

So the easy way out is to remove the package - `sudo apt-get remove resolvconf` and create the file manually - `sudo touch /etc/resolv.conf`. 

Now you can continue using the script the same way as before.

ArchLinux
=========

Thanks to Atul Bhide for these instructions.
Assuming you have logged in as `root`:

1.       Download the latest JDK from oracle. Download the 64 bit JDK only. No need to download or use the 32-bit JDK. It is not needed.

2.       `cd /opt`

3.       `tar xvf /root/Downloads/jdk-7u3-linux-x64.tar.gz`

4.       `ln –s /opt/jdk1.7.0_03 /opt/java`

5.       `cd /usr/lib/Mozilla/plugins`

6.       `ln –s /opt/java/jre/lib/amd64/libnpjp2.so`

7.       Start firefox and log in to rex website. Follow through all the steps till Firefox show the final VPN success screen. This would have created `.juniper_networks` directory.

8.       `vi /etc/pacman.conf` and edit the [multilib] section till it looks like this:

[multilib]

Include = /etc/pacman.d/mirrorlist

9.       `pacman –Syy` = This will update the `multilib` db. Without it you won`t be able to download the 32 bit libs

10.   `pacman –Syu` = Just a precaution but I recommend it.

11.   `pacman –S lib32-gcc-libs lib32-glibc lib32-zlib`

12.   Follow the rest from Kumaresan`s email.

 

Note:

1.       Make sure to do `modprobe tun` else VPN does not work

2.       If you have installed python (3.X) version the python script does not work. So be careful.

3.       You need to install pacman S python2-pip to correctly install the pip package
