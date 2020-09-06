# raspbian-motioneye
Guide for setting up Rasbian lite running MotionEye headless for the Raspberry Pi 4

0. [Preface](#0-preface)
1. [Preparation](#1-preparation)
2. [Setup](#2-setup)
3. [Installation](#3-installation)
4. [Configuration](#4-configuration)
5. [Troubleshooting](#5-troubleshooting)

## 0. Preface

While reading the guide, replace all instances of `<T3xtT0R3pl4ce>` (including the `<` and `>` symbols) with the appropriate value for your usage


## 1. Preparation  

1. Download and install [PuTTY](https://www.PuTTY.org/)
2. Download and install [Rufus](https://rufus.ie/)
3. Download [Raspbian lite](https://www.raspberrypi.org/downloads/raspbian/)
4. Plug in a micro SD card of at least 4GB
5. Run Rufus, select the Raspbian Lite image, select the micro SD card, and click start
6. Open a file explorer and navigate to the micro SD card (if micro SD card doesn't show, see [**Troubleshooting 1**](#1-micro-sd-card-not-showing))
7. Open `config.txt` in an editor and add the following lines to the end of the file:  

	```
	# Use gpu_mem=128 if a camera is attached to the device
	boot_delay=0
	disable_splash=1
	
	gpu_freq_min=250
	arm_freq=1600
	h264_freq=600
	```  
8. Create a new file named `ssh` in the micro SD card's root directory and put a single space in it
9. Create another new file named `wpa_supplicant.conf` and set it's contents to:

	```
	country=<iso alpha-2 code>
	ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
	update_config=1

	network={
		scan_ssid=1
		ssid="<network name>"
		psk="<network password>"
	}
	```
	Valid `ISO Alpha 2` codes examples: `US` `DE` `GB` `CA`
10. Eject the micro SD card from the computer
11. Insert the micro SD card into the Raspberry Pi
12. Plug in the Ethernet cable if available and power on the Raspberry Pi

## 2. Setup
1. Run PuTTY
2. Enter hostname `raspberrypi` and click "Open" (if PuTTY raises error "Host does not exist", see [**Troubleshooting 2**](#2-PuTTY-host-does-not-exist))
3. Login to the shell with login `pi` and password `raspberry`
4. Change password with `passwd` then enter the new password
5. Change root password with `sudo passwd root` the enter the new root password
6. Set a static IP by running `sudo nano /etc/dhcpcd.conf` and adding the following lines:

	```
	interface wlan0
	static ip_address=<static ip>
	static routers=<gateway ip>
	
	interface eth0
	static ip_address=<static ip>
	static routers=<gateway ip>
	```
	Press `Ctrl + O` then `Ctrl + X` to save and exit
7. Update and upgrade software and firmware by running the following commands then reboot:

	```
	sudo apt update
	sudo apt full-upgrade
	sudo rpi-update
	sudo reboot
	```
8. Reconnect to the Raspberry Pi with PuTTY
9. Run `sudo raspi-config` then run `Update` then change netowrk hostname and localization options
10. Reboot and reconnect to the Raspberry Pi with PuTTY using the new hostname/IP address
11. Setup [fail2ban](https://pimylifeup.com/raspberry-pi-fail2ban/) for added security

## 3. Installation
Run the following commands  

```
sudo apt install python-pip python-dev libssl-dev libcurl4-openssl-dev libjpeg-dev libz-dev libavcodec58 libavdevice58 libavformat58 libavutil56 libmariadb3 libmicrohttpd12 libpq5 libswscale5 libwebp6 libwebpmux3
mkdir -p ~/Downloads ; cd ~/Downloads
wget https://github.com/Motion-Project/motion/releases/download/release-4.3.1/pi_buster_motion_4.3.1-1_armhf.deb
sudo dpkg -i pi_buster_motion*
sudo pip install motioneye
```

## 4. Configuration
Run the following commands

```
sudo mkdir -p /etc/motioneye /var/lib/motioneye
sudo cp /usr/local/share/motioneye/extra/motioneye.conf.sample /etc/motioneye/motioneye.conf
sudo cp /usr/local/share/motioneye/extra/motioneye.systemd-unit-local /etc/systemd/system/motioneye.service
sudo systemctl daemon-reload
sudo systemctl enable motioneye
sudo systemctl start motioneye
```

Access MotionEye by going into your browser and going to `<hostname>:8765` or `<raspberry pi ip>:8765`. The default login is `admin` with no password

Configure MotionEye to your liking using `sudo nano /etc/motioneye/motioneye.conf`

Periodically upgrade MotionEye using with the following commands:
 
```
sudo pip install -U motioneye
sudo systemctl restart motioneye
```

## 5. Troubleshooting
### 1. Micro SD card not showing
1. Press `Win` + `R`
2. Type `diskpart` then press `Enter` then run `list disk`
3. Select the appropriate disk with `select disk <x>` where `<x>` is the disk number
4. Run `list partition`
5. Select the appropriate partition with `select partition <y>` where `<y>` is the partition number (usually partition 1, ~256MB)
6. Run `assign letter=i`
7. Check for drive `I` in your file explorer

### 2. PuTTY host does not exist
1. Check router (or use NMap) to get IP address of the Raspberry Pi (sort by access time if needed)
2. Connect to the Raspberry Pi with PuTTY using the IP address
