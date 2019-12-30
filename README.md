# [atet](https://github.com/atet) / [learn](https://github.com/atet/learn) / [**_raspberrypi_**](https://github.com/atet/learn/tree/master/raspberrypi)

[![.img/logo_raspberrypi.png](.img/logo_raspberrypi.png)](#nolink)

# Introduction to Raspberry Pi

**Estimated time to completion: 15 minutes**<br>(excluding waiting times for downloads and updates)

* This introduction to Raspberry Pi only covers what's absolutely necessary to get you up and running
* You are here because **you want to experience a realistic use case for inexpensive single board computers (SBC) while learning fundamental IT skills**
* We will be using a Raspberry Pi Zero W and Bash command line interface (CLI)

--------------------------------------------------------------------------------------------------

## Table of Contents

### Introduction

* [0. Requirements](#0-requirements)
* [1. _Game_ Plan](#1-game-plan)
* [2. Installation](#2-installation)
* [3. Connection](#3-connection)
* [4. Setup](#4-setup)
* [5. Craft Server](#5-craft-server)
* [6. Craft Client](#6-craft-client)
* [7. Next Steps](#7-next-steps)

### Supplemental

* [Why Raspberry Pi?](#why-raspberry-pi)
* [Other Resources](#other-resources)
* [Troubleshooting](#troubleshooting)
* [Acknowledgments](#acknowledgments)

--------------------------------------------------------------------------------------------------

## 0. Requirements

### Software

* This tutorial was developed with Bash on Microsoft Windows 10 with Windows Subsystem for Linux (WSL) using Ubuntu 18.04 LTS
   * WSL is a fully supported Microsoft product for Windows 10, learn how to install it here: [https://docs.microsoft.com/en-us/windows/wsl/install-win10](https://docs.microsoft.com/en-us/windows/wsl/install-win10)
* If you are using MacOS, [your Terminal program is Bash](https://en.wikipedia.org/wiki/Terminal_(macOS))
* Most Linux distributions use or can use Bash; I recommend Ubuntu 18.04 LTS

### Computer Hardware

> [![.img/step00a.png](.img/step00a.png)](#nolink)
> 
> Raspberry Pi Zero W, a single board computer (SBC)

* This tutorial **requires the ~$10 Raspberry Pi Zero W** ("wireless"), you will also need:
   1. Cell phone charger (5V) with micro USB cable
   2. MicroSD card (≥8 GB)
   * NOTE: There exists an older model, Raspberry Pi Zero (without the "W"), that **does not include WiFi functionality required for this tutorial**
* This tutorial should also work with any computer that can run Linux

### WiFi Network

**The Pi Zero W requires very specific WiFi settings**

1. You must be able to connect a new wireless device to your WiFi network using only the network name (a.k.a. SSID) and network password
   * Some networks may require additional registration for new devices, like a school or public hotspot; consult your IT department about adding your Pi
   * The network can be hidden
2. The wireless network **must have disabled** [**"wireless isolation"** (a.k.a. AP isolation, station isolation, or client isolation)](https://www.howtogeek.com/179089/lock-down-your-wi-fi-network-with-your-routers-wireless-isolation-option/)
   * There is no compromise on this one; if wireless isolation is not disabled on your network, you will not be able to connect to your Pi trhough WiFi

[Back to Top](#table-of-contents)

--------------------------------------------------------------------------------------------------

## 1. _Game_ Plan

* Let's do something fun and setup a Craft<sup>[[1]](#acknowledgments)</sup> server to host multiple players at the same time
* Several different skills are introduced in this short project:
   1. Basic IT: Formatting disks and installing operating systems
   2. Networking: Local area networking and secure shell access
   3. System Administration: Command line interface, building from source code, and running a server

> [![.img/step01a.png](.img/step01a.png)](#nolink)
> 
> Messing around in Craft on single player mode

[Back to Top](#table-of-contents)

--------------------------------------------------------------------------------------------------

## 2. Installation

### 2.1. Download Operating System Image

* We are using **Raspbian Linux Lite**, which is command line interface (CLI) only
* **You don't need prior CLI experience for this tutorial**
* Download the latest image from: [https://www.raspberrypi.org/downloads/raspbian/](https://www.raspberrypi.org/downloads/raspbian/)
   * Make sure you download the "Lite" version
   * Extract the ZIP folder once the image is downloaded

[![.img/step02a.png](.img/step02a.png)](#nolink)

### 2.2. Prepare Micro SD Card

**This is the only step that can be frustrating and may take a few retries**

1. Format micro SD card
   * Windows Disk Management: Press the Windows key and search for "disk management"
2. Reformat the card
   * Windows: Creating a new simple disk in Disk Management here will also reformat the SD card
   * (TODO: More details)
3. Burn the Raspbian image onto the formatted micro SD card
   * I used Rufus Portable v3.8: [https://rufus.ie/](https://rufus.ie/)
   * If you do not format the SD card **before** this step (in case you used a utility other than Disk Management), you may not be able to read the SD card as the `boot` drive
   * When the image burn is complete, just cancel out of Rufus

[![.img/step02b.png](.img/step02b.png)](#nolink)

4. Access the new `boot` drive
   * **If you cannot read the card**; repeat steps 1-3
   * Show file name extensions

[![.img/step02c.png](.img/step02c.png)](#nolink)

5. Make two new configuration files
   * Make a new file called `ssh` (no file extension)
      * This file will remain empty
   * Make a new file called `wpa_supplicant.conf`

[![.img/step02d.png](.img/step02d.png)](#nolink)

6. `wpa_supplicant.conf` settings
   * Open `wpa_supplicant.conf` with Notepad (right-click → Open with → Choose another app → Notepad)

   [![.img/step02e.png](.img/step02e.png)](#nolink)

   * Copy and paste the following, changing `<NETWORK NAME>` and `<NETWORK PASSWORD>` to match your network's and save
      * If your WiFi network is hidden, you must use the line `scan_ssid=1`
      * Change the country code as appropriate

   ```
   ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
   update_config=1
   country=US

   network={
      ssid="<NETWORK NAME>"
      scan_ssid=1
      psk="<NETWORK PASSWORD>"
      key_mgmt=WPA-PSK
   }
   ```

### 2.3. Headless OS installation

**Headless** means that we won't attach a monitor or keyboard to the Pi and all communication to it will be through command line interface (CLI) from another computer

1. The Pi should not be powered on at this time
2. Insert prepared micro SD card
3. Plug the micro USB power into the Pi's power port and the green LED should start blinking
   * Coffee Break: It'll take ~10 mins while the Raspbian OS installs, configures itself, and automatically connects to your WiFi network
4. When the Pi is ready, the green LED should stop blinking and stay on

[Back to Top](#table-of-contents)

--------------------------------------------------------------------------------------------------

## 3. Connection

### 3.1. Determining the IP address of your headless Raspberry Pi

**This may be tricky depending on your unique situation:**

1. If you have administrative access to your local network's router:
   * Log into the router and determine the IP address that corresponds to the hostname `raspberrypi` (default name for your Pi)
   * Below is an example router administration page to look up the IP addresses associated with connected devices

   [![.img/step031a.png](.img/step031a.png)](#nolink)

2. If you **do not** have admin access to your network's router, you can try:
   * Scanning your network for connected IP addresses: https://stackoverflow.com/a/23432113
   * Connecting using Apple Bonjour and `pi@raspberrypi.local`: https://raspberrypi.stackexchange.com/a/45199

3. If the above fails, check the [Troubleshooting](#troubleshooting) section

### 3.2. Remote connection to Raspberry Pi

* We will use Bash secure shell (SSH) through the WSL command line interface
* Knowing the Pi's IP address, SSH as the default username `pi@<IP ADDRESS>` and password `raspberry`
   * Typing the password will be invisible for security
   * You may get a warning of `The authenticity of host...`, just answer `yes`

   [![.img/step03c.png](.img/step03c.png)](#nolink)

**If you successfully log in, all the hard work is done**

[Back to Top](#table-of-contents)

--------------------------------------------------------------------------------------------------

## 4. Setup

### 4.1. Add update source

* After logging in, we need to add a location where the Pi will look for updates
* Copy and paste the line code after the "`$`" and press ENTER:

```
$ sudo nano /etc/apt/sources.list
```

* Uncomment (remove "`#`" in front of): `deb-src http://raspbian...`
   * Press `CTRL+O` then `ENTER` to save
   * Press `CTRL+X` to exit

### 4.2. Updates

* We need to update everything on the Pi and install a few new programs that the Craft server is dependent on (a.k.a. dependencies)
* Copy and paste the entire multi-line block of code after the "`$`" and press ENTER:
   * Coffee break #2: It'll take 10+ mins. and you don't need to babysit this

```
$ sudo apt-get update && \
  sudo apt-get -y upgrade && \
  sudo apt-get -y install git python-pip cmake libglew-dev xorg-dev libcurl4-openssl-dev && \
  sudo apt-get -y build-dep glfw && \
  python -m pip install requests
```

[Back to Top](#table-of-contents)

--------------------------------------------------------------------------------------------------

## 5. Craft Server

### 5.1. Download Craft server

* Download the files for running a Craft server from GitHub (~15 MB)

```
$ cd ~ && \
  git clone https://github.com/fogleman/Craft.git && \
  cd ~/Craft && \
  cmake .
```

### 5.2. Build Craft program

* We can now build the Craft program to run on the specific Raspberry Pi Zero W hardware
   * Coffee break #3: It'll take ~10 mins. and you don't need to babysit this

```
$ make && \
  gcc -std=c99 -O3 -fPIC -shared -o world -I src -I deps/noise deps/noise/noise.c src/world.c
```

### 5.3. Start Craft server

* The server program hosts a persistent, shared world for users (clients) to connect to and play together
* Once you run the line below, the server program will start displaying events as they happen in the game world (players connecting, logging out, etc.)

```
$ python server.py
```

* Remember the IP address for the Pi, this IP is the address that clients will connect to within your local area network (LAN)

[Back to Top](#table-of-contents)

--------------------------------------------------------------------------------------------------

## 6. Craft Client

### 6.1. Registering an account

* Even though we made our own sever here, we still need to register an account on the creator of Craft's website: https://craft.michaelfogleman.com/

> "Why register?
>
> You can play on most game servers anonymously. **However, without registering you will not be able to make changes in most areas of the world**."

* After you register and verify your email address, log back into https://craft.michaelfogleman.com/ and make an Identity Token, it should look similar to:

```
/identity <USERNAME> 0123456789abcdef0123456789abcdef
```

[![.img/step06a.png](.img/step06a.png)](#nolink)

* **Highlight the line and `CTRL`+`C`** (pressing the copy to clipboard button didn't work for me to paste in the game)
* NOTE: You can only see this key once; if you close this window, you will have to make another key if you need it again

### 6.2. Download Craft client

* Download the Craft client for Windows or MacOS here: https://www.michaelfogleman.com/projects/craft/
* This is a "portable" program; nothing needs to be installed, just extract the ZIP file
* Run `craft.exe`

### 6.3. Connecting to Craft server

* Once the game starts, press "`T`" and `CTRL`+`V` to paste in your Identity Token and press ENTER (slash "`/`" in front is used to denote system commands)
* Press "`T`" and enter "`/online <SERVER IP>`" to connect to your Raspberry Pi Zero Craft server
   * Once on the server, you should automatically be connected as your account
   * The server may accidentally log you off your username after you connect; if it says you are a "guest", you must re-login: "`/login <USERNAME>`"

[![.img/step06b.png](.img/step06b.png)](#nolink)

### 6.4. Playing Craft

**CONGRATS! You're done with the tutorial**

Have some fun and build a new world in your own Craft playground:

Button | Action
--- | ---
`W`, `A`, `S`, `D` | Movement
`Left-Mouse` | Destroy block
`Right-Mouse` | Create block
`Mouse-Wheel` | Cycle through block types
`CTRL`+`Right-Mouse` | Create light source
`T` | Chat
`/` | Command: Chat but with `/` added
`ESC` | Mouse control back to OS (to close or maximize/minimize Craft window)
More controls | https://github.com/fogleman/Craft#controls

### 6.5. Multiplayer vs. single-player

[![.img/step06c.png](.img/step06c.png)](#nolink)

* You can have a single-player experience in Craft without connecting to a server and have your world saved locally on your computer
* Other players will not be able to connect to your world in single-player mode
* The only way to have multiplayer in Craft is to connect to a server that is running the Craft server program

### 6.6. Cleanup

**After you are done with this tutorial:**

* You can continue to have the server program run as long as you want to play on it
* You can shut down the server program by pressing `CTRL+C`, then shut down the Pi by executing:

```
$ sudo shutdown -h now
```

[Back to Top](#table-of-contents)

--------------------------------------------------------------------------------------------------

## 7. Next Steps

**We touched on a bunch of different IT tasks here; you're easily on your way to becoming a self-sufficient ["techie"](https://www.merriam-webster.com/dictionary/techie), it just takes a lot of experimenting and completing projects like this**

* Have other people join in at your home network!
   * What we've setup here, only computers within your local area network (LAN) can connect to your server
* Learn how to make cloud instances and share your server to the world
   * WARNING: You should "harden" your server security first before exposing any of your servers to the public internet: [https://www.upguard.com/blog/10-essential-steps-for-configuring-a-new-server](https://www.upguard.com/blog/10-essential-steps-for-configuring-a-new-server)
* Now that you already have a Raspberry Pi, try some other fun projects: https://projects.raspberrypi.org/en/
* Trying to go back to sleep at 3AM? Read the official Raspberry Pi starter guide: https://projects.raspberrypi.org/en/pathways/getting-started-with-raspberry-pi

**If you would like to learn more about Bash and command line interface (CLI), please see [Atet's 15 Minute Introduction to Regular Expressions (in Bash)](https://github.com/atet/learn/blob/master/regex/README.md#atet--learn--regex)**

[Back to Top](#table-of-contents)

--------------------------------------------------------------------------------------------------

## Why Raspberry Pi?

* **_Trust me on this one_**: There's a huge difference in your learning experience when the brand you're working with has sold 10+ million computers vs. lesser-known alternatives that may be a _bit_ cheaper
* With a larger userbase, there's reliable community and official support, bugs get fixed quicker, you can expect timely updates, etc.
* The Raspberry Pi Zero is amazing for its price point, [but it's not going to play Crysis](https://en.wikipedia.org/wiki/Crysis_(video_game)#Legacy); this brand has other more powerful and more expensive computers if you need the horsepower

> [![.img/wrpa.png](.img/wrpa.png)](#nolink)
>
> Raspberry Pi 4 Model B

[Back to Top](#table-of-contents)

--------------------------------------------------------------------------------------------------

## Other Resources

Description | Link
--- | ---
Official Raspberry Pi Help | https://www.raspberrypi.org/help/
Official Craft Server Installation | https://github.com/fogleman/Craft#linux-ubuntu
Official Raspberry Pi Getting Started Guide | https://projects.raspberrypi.org/en/pathways/getting-started-with-raspberry-pi
Official Raspberry Pi Project Ideas | https://projects.raspberrypi.org/en/

[Back to Top](#table-of-contents)

--------------------------------------------------------------------------------------------------

## Troubleshooting

Issue | Solution
--- | ---
I don't need a password to use my WiFi | If you administer your own "open" network that does not require a password, **you should secure your network ASAP**: [https://lifehacker.com/how-to-make-your-wifi-router-as-secure-as-possible-1827695547](https://lifehacker.com/how-to-make-your-wifi-router-as-secure-as-possible-1827695547)
Cannot access micro SD card `boot` drive after OS image burn | Must have formatted the card (to have a partition) **BEFORE** burning the Raspbian image to the card; repeat steps 1-3 in [2.2. Prepare Micro SD Card](#22-prepare-micro-sd-card)
Reformatted micro SD card's free space is **less** than before | Multiple partitions may exist on card from a previous OS burn; erase all partitions before formatting as **one partition**
Cannot find Pi's IP address | _Did the Pi actually connect to your WiFi network successfully?_ Go back and verify `wpa_supplicant.conf` or confirm you have permission to connect new devices to your network<br><br>You can also verify connection status and the assigned IP address by connecting to the Pi _locally_ (see below)  
Player camera slowly looks up automatically | Bug? I had this issue when the game is maximized to fullscreen or the window was expanded outside the limits of the screen; just play windowed if this happens

### **Advanced methods to connect to a headless Raspberry Pi**

### Connecting via USB cable

* You will need the USB charging cable
* https://www.tomshardware.com/reviews/raspberry-pi-headless-setup-how-to,6028.html

### Connecting locally

If you **do not** have access to your network's router but have USB and HDMI adapters:
   * You need an HDMI compatible TV or monitor
   * Adapters for micro USB (Male) to USB-A (Female) and Mini HDMI (Male) to HDMI (Female)

   [![.img/step03a.png](.img/step03a.png)](#nolink)

   * Connect a USB keyboard and HDMI monitor to the Pi using the adapters
   * Login as default username `pi` and password `raspberry`
   * Execute `ifconfig` and determine Pi's IP address
      * Look under the `wlan0` section
      * Find the IP address after `inet`
      * If the Pi did not successfully connect to your network, there will not be an IP address shown

   [![.img/step03b.png](.img/step03b.png)](#nolink)

[Back to Top](#table-of-contents)

--------------------------------------------------------------------------------------------------

## Acknowledgments

1. Craft, the free, open-source Minecraft clone by Michael Fogleman: <a href="https://www.michaelfogleman.com/projects/craft/" target="_blank">https://www.michaelfogleman.com/projects/craft/</a>

[Back to Top](#table-of-contents)

--------------------------------------------------------------------------------------------------

<p align="center">Copyright © 2019-∞ Athit Kao, <a href="http://www.athitkao.com/tos.html" target="_blank">Terms and Conditions</a></p>