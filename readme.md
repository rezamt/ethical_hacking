# Network Hacking

## Install the driver for wireless network card

The RTL8812AU chipset requires a specific driver, and check if the driver is installed using the following command.

```bash
lsmod | grep 8812au
```

If the driver is not installed, install them.

```bash
sudo apt update
sudo apt install dkms build-essential linux-headers-$(uname -r) git

git clone https://github.com/aircrack-ng/rtl8812au.git
cd rtl8812au

```

## Normal Mode vs Monitor Mode

Monitor mode allows your interface to get packets that are not meant to be sent to you in the network.

```bash
# Kill the network
airman-ng check kill

# Set the interface to monitor mode
sudo iwconfig wlan0 mode monitor
```

## Manually Set Mac Address & Mac Address Reset Error

Use the following commands to manually set the Mac Address for the wireless interface.

> ifconfig is old, the recommended way is `ip` command.

```bash
sudo ifconfig wlan0 down

sudo ifconfig wlan0 hw ether 00:11:22:33:44:55

sudo ifconfig wlan0 up
```

If using the newer `ip` command, we can do:

```bash
ip link # check the wireless interface
sudo ip link set wlan0 down

sudo ip link show
sudo ip link set wlan0 00:11:22:33:44:55

sudo ip link set wlan0 up
```

```bash
sudo make dkms_install
```

Add the following two lines at the end of `/etc/NetowrkManager/NetworkManager.conf` file to prevent the Network Manager from resetting MAC address.

```bash
[device]
wifi.scan-rand-mac-address=no

[connection]
wifi.cloned-mac-address=preserve
```
