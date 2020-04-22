# raspbian-motioneye
Guide for setting up Rasbian lite running MotionEye headless for Raspberry Pi 4

0. [Preface](## 0. Preface)
1. [Preparation](## 1. Setup)
2. [Setup](## 2. Setup)
3. [Installation](## 3. Installation)
4. [Configuration](## 4. Configuration)

## 0. Preface

While reading the guide, replace all instances of `<T3xtT0R3pl4ce>` with the appropriate value for your usage, including the `<` and `>` symbols 


## 1. Setup  

1. Download and install [Putty](https://www.putty.org/)
2. Download and install [Rufus](https://rufus.ie/)
3. Download [Raspbian lite](https://www.raspberrypi.org/downloads/raspbian/)
4. Plug in a micro SD card of at least 4GB
5. Run Rufus, select the Raspbian Lite image, select the micro SD card, and click start
6. Open a file explorer and navigate to the micro SD card
<details><summary>Click if micro SD card not showing</summary>
Press `Win` + `R`. Type `diskpart` then press `Enter` then run `list disk`. Select the appropriate disk with `select disk X` where X is the disk number. Run `list partition`. Select the appropriate partition with `select disk Y` where Y is the partition number (usually partition 1, ~256MB). Run `assign letter=i`.</details>
7. Open `config.txt` in an editor and add the following lines to the end of the file:  

	```  
	gpu_mem=16
	boot_delay=0
	disable_splash=1
	
	arm_freq=1600
	h264_freq=600
	gpu_freq_min=250
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
10. Eject the micro SD card from the computer
11. Insert the micro SD card into the Raspberry Pi
12. Plug in the Ethernet cable if available and power on the Raspberry Pi