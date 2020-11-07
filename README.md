# arm64 Debian - ADSB Receiver Setup for FlightAware

#### 1.)  Follow the instructions in the file `arm64 Debian Basic Install for Rock64` or `arm64 Debian Basic Install for PineH64B`, a 16GB eMMC module is sufficient as the whole setup requires only 2.5GB in space when finished.

#### 2.)  Install sudo    (log-in as root)

`apt install sudo`

`nano /etc/sudoers` scroll down to `User privilege specification` and copy `root` for your specific username and save

#### 3.)  Reduce network speed to 100MBit and set network to static IP address

`sudo apt install ethtool`

`sudo nano /etc/network/interfaces`

  # This file describes the network interfaces available on your system
  # and how to activate them. For more information, see interfaces(5).
  
  source /etc/network/interfaces.d/*
  
  # The loopback network interface
  
  auto lo
  iface lo inet loopback
  
  # The primary network interface
  auto eth0
  allow-hotplug eth0
  iface eth0 inet static
            link-speed 100
            link-duplex full
            ethernet-autoneg off
            address 192.168.1.xxx
            netmask 255.255.255.0
            gateway 192.168.1.xxx
            dns-nameservers 192.168.1.xxx
 
replace xxx with your relevant/desired subaddress
