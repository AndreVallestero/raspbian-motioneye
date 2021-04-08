# raspbian-motioneye
Guide for setting up Rasbian lite running MotionEye headless for the Raspberry Pi 4

0. [Preface](#0-preface)
1. [Preparation](#1-preparation)
2. [Setup](#2-setup)
3. [Installation](#3-installation)
4. [Configuration](#4-configuration)
5. [HTTPS / TLS (Optional)](#5-https--tls-optional)
6. [Troubleshooting](#6-troubleshooting)

## 0. Preface

While reading the guide, replace all instances of `<T3xtT0R3pl4ce>` (including the `<` and `>` symbols) with the appropriate value for your usage

If you would like to boot off a USB, HDD, or SSD, follow [this](https://www.tomshardware.com/how-to/boot-raspberry-pi-4-usb) guide (use [gnu] dd rescue to clone the sdcard) then use `fdisk` delete + new (make sure to start new partition on same sector as before) then `e2fsck` and `fsresize`, all while being on the sdcard, then reboot.

It's highly recommended that you setup https with DDNS (afraid org) + Lets Encrypt (certbot) if you would like to access the web client remotely.

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
8. If using ssh, create a new file named `ssh` in the micro SD card's root directory and put a single space in it
9. If using wifi, create another new file named `wpa_supplicant.conf` and set it's contents to:

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
4. Change root password with `sudo passwd root` the enter the new root password
5. Create a new user `sudo adduser alice`
6. Add user to sudo group `sudo usermod -a -G sudo [new user name]`
7. Switch to new user `sudo su -  [new user name]`
8. Stop user "pi" processes `sudo pkill -u pi`
9. Delete the "pi" user `sudo deluser -remove-home pi`
10. Set a static IP by running `sudo nano /etc/dhcpcd.conf` and adding the following lines (can be skiped with static dhcp on your router):

	```
	interface wlan0
	static ip_address=<static ip>
	static routers=<gateway ip>
	
	interface eth0
	static ip_address=<static ip>
	static routers=<gateway ip>
	```
	Press `Ctrl + O` then `Ctrl + X` to save and exit
11. Update and upgrade software and firmware by running the following commands then reboot:

	```
	sudo apt update
	sudo apt full-upgrade
	sudo rpi-update
	sudo apt autoremove --purge
	sudo apt clean
	sudo reboot
	```
12. Reconnect to the Raspberry Pi with PuTTY
13. Run `sudo raspi-config` then run `Update` then change netowrk hostname and localization options
14. Reboot and reconnect to the Raspberry Pi with PuTTY using the new hostname/IP address
15. Setup [fail2ban](https://pimylifeup.com/raspberry-pi-fail2ban/) for added security

## 3. Installation
Run the following commands  

```
sudo apt install python-pip python-dev libssl-dev libcurl4-openssl-dev libjpeg-dev libz-dev libavcodec58 libavdevice58 libavformat58 libavutil56 libmariadb3 libmicrohttpd12 libpq5 libswscale5 libwebp6 libwebpmux3 ffmpeg
mkdir -p ~/Downloads ; cd ~/Downloads
wget https://github.com/Motion-Project/motion/releases/download/release-4.2.2/pi_buster_motion_4.2.2-1_armhf.deb
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

## 5. HTTPS / TLS (Optional)
1. Acquire a [sub]domain via https://freedns.afraid.org/ (free) or some other domain / DNS service and point it to your IP (DDNS is recommended instead)

2. Run the following commands

```
sudo apt install nginx certbot python-certbot-nginx
sudo rm /etc/nginx/sites-available/default
sudo rm /etc/nginx/sites-enabled/default
sudo nano /etc/nginx/conf.d/<my.d0m41n.0rg>.conf
```

3. Insert the following lines

```
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name <my.d0m41n.0rg>;
    location / {
        proxy_pass http://127.0.0.1:8765/;
    }
}
```

4. Portforward 80 to 80 on your [sub]domain aquired in step 5.1

5. Run the follow commands

```
sudo systemctl enable nginx
sudo systemctl start nginx
sudo certbot --nginx
```

6. Disable portforward for port 80 to 80, and enable it for port 443 to 443


## 6. Troubleshooting
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
