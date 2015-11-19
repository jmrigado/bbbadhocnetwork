# Beaglebone Black as an Ad-Hoc Network
Instructions on how to use a Beaglebone Black(BBB) as an Ad-Hoc network server using the EDIMAX USB dongle.   Detailed BBB usage information can be viewed at http://beagleboard.org

## Before we start
* Install the latest Debian release for the BBB found here: http://beagleboard.org/latest-images
* Connect you computer to the BBB using either the USB or CAT5 solutions
* SSH to the BBB
* Get the remaining tools needed to run the solution on the Debian install:

```
sudo apt-get install bridge-utils
sudo apt-get install hostapd
```

##Get the drivers required for the EDIMAX USB dongle
* You can pull the required version from the Realtek website: http://www.realtek.com.tw/downloads/downloadsView.aspx?Langid=1&PNid=21&PFid=48&Level=5&Conn=4&DownTypeID=3&GetDown=false
* Copy the file to the BBB
* Use the terminal to finish creating the required driver:
```
unzip 0001-RTL8188C_8192C_USB_linux_v4.0.2_9000.20130911.zip (or whatever you .ZIP file is called)
cd <new_unzipped_dir>/wpa_supplicant_hostapd
tar xf wpa_supplicant_hostapd-***.tar.gz
cd wpa_supplicant_hostapd-***/hostapd
make
make install
cd ..
```

##Install a new version of the HOSTAPD tool
Again use the terminal to enter the following commands
```
sudo mv /usr/sbin/hostapd /usr/sbin/hostapd.bak
sudo mv hostapd /usr/sbin/hostapd.edimax 
sudo ln -sf /usr/sbin/hostapd.edimax /usr/sbin/hostapd 
sudo chown root.root /usr/sbin/hostapd 
sudo chmod 755 /usr/sbin/hostapd
```

##Configuring the network

###Network Interfaces
In the terminal use your favorite editor to open the interfaces file
```
sudo nano /etc/network/interfaces
```

Now we need to configure the network so enter the following into the editor comment out (#) anything that exists in their already
```
#loopback adapter
auto lo
iface lo inet loopback
#wired adapter
iface eth0 inet dhcp
#bridge
auto br0
iface br0 inet dhcp
bridge_ports eth0 wlan0
```
###HOSTAPD.CONF
Now we need to update the HOSTAPD configuration, so lets load the configuration file
```
sudo nano /etc/hostapd/hostapd.conf
```
We can then replace the contents with the following... ensure that the driver matches the one that you configured in the earleir section on downloading a driver.  Also ensure you provide your own network SSID and passphrase.
```
interface=wlan0
driver=rtl871xdrv
bridge=br0
ssid=<your SSID name here>
channel=1
wmm_enabled=0
wpa=1
wpa_passphrase=<your password here>
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
auth_algs=1
macaddr_acl=0
```

##Finishing Up
###Testing###
Almost done... we now need to do a quick reboot
```
sudo reboot
```
You can now test your configuration using the following command
```
sudo hostapd -dd /etc/hostapd/hostapd.conf
```
If it runs as expected, you can kill the running process and add it to startup by editing
```
sudo nano /etc/default/hostapd
```
and uncommenting and updating the following line
```
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```

###DHCP Server###
If all is workign we need to edit the DHCP configuration

```
file: /etc/udhcpd.conf
```
Add the following configuration(you cna configure the start-end to be what ever you want jsut remember to keep it consistent with previous IP information specified previously:
```
start      192.168.2.1
end        192.168.2.5
interface  wlan0
max_leases 5
option subnet 255.255.255.252
```

Start it all up...
Create a shell script (mine is startup.sh) with the following entries:
```
echo APD ifconfig
ifconfig wlan0 192.168.2.2
echo APD starting dhcp server
udhcpd
echo APD start wifi AP
hostapd -dd /etc/hostapd/hostapd.conf
```
Save and...

Set permissions
```
chmod 700 startup.sh
```

Execute
```
./startup.sh
```

Now take a second device and connect to the network you just created!

