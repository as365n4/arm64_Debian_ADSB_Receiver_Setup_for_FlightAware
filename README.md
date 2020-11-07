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
 
replace `xxx` with your relevant/desired subaddress

`sudo nano /etc/resolv.conf`

`nameserver 192.168.1.xxx`

`xxx` should match your DNS Server

#### 4.)  Install system monitoring

`sudo apt install hddtemp lm-sensors glances htop screenfetch`

`sudo nano /etc/glances/glances.conf`     comment / uncomment various sections as required

`sudo nano /lib/systemd/system/glances.service`

    [Unit]
    Description=Glances
    After=network.target
    
    [Service]
    ExecStart=/usr/bin/glances -w
    Restart=on-abort
    
    [Install]
    WantedBy=multi-user.target

`sudo systemctl status glances.service`   check that glances is running and pressing ‘q’ returns to console

`sudo reboot`     to restart the machine and all systemd services, once machine is up again check `192.168.1.xxx:61208` if glances is running correctly.

`senors`      shows all data gathered by lm-sensors

`glances`     shows a variety of system/machine data which can be configured by changing `/etc/glances/glances.conf`

#### 5.)  Install GPS Software

`sudo apt install gpsd`

`sudo dpkg-reconfigure gpsd`

`sudo nano /lib/systemd/system/gpsd.socket`     change `ListenStream=127.0.0.1:2947` to `ListenStream=0.0.0.0:2947`

`sudo nano /etc/default/gpsd`

    START_DAEMON=”true”
    USBAUTO=”true”
    DEVICES=”/dev/ttyXYZ”
    GPSD_OPTIONS=”-n”
    GPSD_SOCKET=”/var/run/gpsd.sock”
 
Check with `ls /dev/` for correct device and replace `XYZ` with `AMA0` or `ACM0` or `USB0` as appropriate.

`sudo systemctl enable gpsd`

`sudo systemctl start gpsd`

`sudo systemctl status gpsd`

`sudo reboot`

Use `cgps` or `gpsmon` to check GPS data and position. (pressing ‘q’ returns to console)

#### 6.)  Install FlightAware Software

`sudo apt install git debhelper librtlsdr-dev pkg-config dh-systemd libncurses5-dev libbladerf-dev libhackrf-dev liblimesuite-dev tcl8.6-dev python3-dev python3-venv libz-dev libboost-system-dev libboost-program-options-dev libboost-regex-dev libboost-filesystem-dev`

`makedir flightaware`

`cd flightaware`

`git clone https://github.com/flightaware/dump1090.git`

`cd dump1090`

`sudo dpkg-buildpackage -b --no-sign`

`cd ..`

`sudo apt install /home/user/flightaware/dump1090-fa_X.0_arm64.deb`

`sudo apt install /home/user/flightaware/dump1090-fa-dbgsym_X.0_arm64.deb`

`sudo systemctl enable dump1090-fa`

`sudo systemctl start dump1090-fa`

`sudo systemctl status dump1090-fa`

`git clone https://github.com/flightaware/piaware_builder.git`

`cd piaware_builder`

`./sensible-build.sh buster`

`cd package-buster`

`sudo dpkg-buildpackage -b --no-sign`

`sudo apt install /home/user/flightaware/piaware_builder/piaware_X.0_arm64.deb`

`sudo apt install /home/user/flightaware/piaware_builder/piaware-dbgsym_X.0_arm64.deb`

`sudo systemctl enable piaware`

`sudo systemctl start piaware`

`sudo systemctl status piaware`

`sudo reboot`

`sudo piaware-config allow-auto-updates yes`

`sudo piaware-config allow-manual-updates yes`

`sudo piaware-config feeder-id XXXXX`     replace X with Unique Identifier found on FlightAware web site (your personal account).

#### 7a.) Optional, install tar1090 web interface

`sudo bash -c “$(wget -q -O – https://raw.githubusercontent.com/wiedehopf/tar1090/master/install.sh)”`

`sudo nano /usr/local/share/tar1090/html/config.js`     Amend `web interface` if needed

#### 7b.) Optional, install Dump1090-OpenLayers3 mod

`git clone https://github.com/alkissack/Dump1090-OpenLayers3-html.git`

`cd /usr/share/dump1090-fa`

`sudo cp -R html original-html`

`sudo rm -R -f html`

`cd /home/user/Dump1090-OpenLayers3-html`

`sudo cp -R public_html /usr/share/dump1090-fa/html`

`cd /usr/share/dump1090-fa/html`

`sudo nano config.js`     Amend `DefaultZoomLvl`, `SiteLat`, `SiteLon`, `ShowMouseLatLong`, `ShowMaxRange`.

`sudo cp -R config.js /usr/share/dump1090-fa/config.js`

`sudo reboot`

#### 8.)  Console auto log-in and Glances start at boot/log-in

`nano /home/user/.bashrc`     add `glances` to the END of file

`sudo nano /etc/systemd/logind.conf`      uncomment `#NAutoVTs=6` and set to `NAutoVTs=2`

`sudo mkdir /etc/systemd/system/getty@tty1.service.d`

`sudo nano /etc/systemd/system/getty@tty1.service.d/override.conf`

    [Service]
    ExecStart=
    ExecStart=-/usr/sbin/agetty --autologin user --noclear %I $TERM

`sudo systemctl enable getty@tty1`

`sudo reboot`

Done, enjoy your new ADSB receiver !
