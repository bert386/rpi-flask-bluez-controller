# Elev-air-Hardware

## Assembleing components

First lets plug Grove Base Hat to Pi Zero W.
And then, plug modules as follows.

1. Grove MQ2 Gas sensor - A0 of base hat
2. Grove RTC module - I2C slot of base hat
3. Grove Temp/Humi module - D5 of base hat
4. Grove 2 SPTD relay module - D16 of base hat

## Configuration

- Installation of Raspberry Pi OS

  **1. Flash latest [Raspberry Pi OS (32-bit)](https://downloads.raspberrypi.org/raspios_full_armhf_latest) to sd card**

  _There are several tools to flash sd card with raspberry pi system image such as Win32 Disk Imager etc._

  **2. Enable ssh by generating empty file (**_File name: ssh_**) in root of SD card**

- Pi Zero USB/Ethernet Gadget

  _In order to install packages and configure system, we could use Pi Zero based USB/Ethernet Gadget functionality.
  By this, you could plug Pi Zero module via USB to your machine and configure via SSH easily._

  **1. EDIT THE CONFIG.TXT AND CMDLINE.TXT FILES**

  Once that’s finished, navigate to the root directory of the micro SD card (i.e. boot (E:)) and open the file config.txt with Notepad etc. Using Notepad to edit the file will make it hard to tell where the line breaks are, and this file uses line breaks to separate the commands. Scroll down to the bottom of the file and add **dtoverlay=dwc2** to a new line:

  ```
    [pi4]
    # Enable DRM VC4 V3D driver on top of the dispmanx display stack
    dtoverlay=vc4-fkms-v3d
    max_framebuffers=2

    [all]
    #dtoverlay=vc4-fkms-v3d

    # Added line
    dtoverlay=dwc2
  ```

  Save and exit the config.txt file.

  Now open the cmdline.txt file with Notepad. Make sure “Word Wrap” is off to view the file as a single line. You don’t want to add any line breaks to this file. All of the commands need to be in a single line. If you use Notepad to edit this file, you may create unintentional line breaks, so using Notepad is best. Scroll across the line and find the command **rootwait**. After the **rootwait** command, add **modules-load=dwc2,g_ether**

  Save and exit the cmdline.txt file.
  Now you can eject the SD card from your computer.
  Insert your SD card into the Pi Zero, and plug your micro USB to USB adapter (or cable) into the Pi’s micro USB port (not the power connection), and the other end into your computer:

  ![USBZeroW](images/PiZeroWUSB.jpg)

  You’ll see that Windows recognizes the Pi, and will attempt to setup the device.
  At this point, open PuTTY and try SSHing into the Pi with the address raspberrypi.local:

  ![PiSSH](images/PiSSH.png)

  If you can log in to your Pi at this point, that’s good. You can skip the next section, "Install the RNDIS Drivers" and go to the "Setting Up Shared Internet Access" further below.

  If you do get an error message saying something like “Unable to open connection to raspberrypi.local. Host does not exist”, you’ll just need to install the RNDIS drivers on your computer. I’ll explain how to do this in the next section:

  **2. INSTALL THE RNDIS DRIVERS**

  With your Pi Zero still connected to your computer, navigate to the Windows Device Manager. Under “Other devices” find "RNDIS/Ethernet Gadget", and right click on it. Then click “Update driver software” from the menu:

  **3. SETTING UP SHARED INTERNET ACCESS**

  To share your computer’s internet connection with the Pi, we need to allow network sharing on one of your computer’s network connections.

  With your Pi Zero plugged in and powered on, navigate to the “Network Connections” window:

  Your Pi will show up as a separate network connection. It will say “RNDIS/Ethernet Gadget” under the connection name. In my case, it’s “Ethernet 2”.

  Now you should decide which connection you want your Pi to access the Internet over (i.e. WiFi or Ethernet). I want my Pi to access the internet over my computer’s WiFi connection, so I’ll need to enable my WiFi connection to allow sharing. Right click on the connection you want to use, then select “Properties”.

  In the WiFi Properties window, click on the “Sharing” tab.
  Click the box that says “Allow other network users to connect through this computer’s Internet connection”, then click the drop down menu below that. Find the network connection given to your Pi (Ethernet 2 in my case), select it, then click OK.

  The network you chose to share access with your Pi will show the name of your WiFi network, followed by “Shared” in the “Network Connections” window :

  Now reboot your Pi and log back into it with PuTTY.

  Enter ifconfig at the command prompt, and you should see a usb0 connection showing TX and RX packets being sent and received:

## Setting up WiFi hotspot

**In ssh terminal, install packages**

```
sudo apt-get update
sudo apt-get upgrade
sudo apt install hostapd -y
sudo systemctl unmask hostapd
sudo systemctl enable hostapd
sudo apt install dnsmasq -y
sudo DEBIAN_FRONTEND=noninteractive apt install -y netfilter-persistent iptables-persistent
```

**Assign a Static IP Address**

To start, run the following command in the Terminal:

```
sudo nano /etc/dhcpcd.conf
```

It will open the configuration file for dhcpcd. Scroll to the bottom of this file and add the following lines:

```
interface wlan0
static ip_address=192.168.4.1/24
nohook wpa_supplicant
```

Save your changes by pressing Ctrl + O followed by Ctrl + x.

**Configure Your DHCP and DNS Services**

The dnsmasq package provides a default configuration file, but we don’t need all the options included in this file.

To make things easier, rename dnsmasq’s default configuration file and create a replacement file that’s completely empty. Then open this new “dnsmasq.conf” file in the Nano text editor and add only the configuration options that we actually need.

To start, run the following Terminal commands:

```
sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig
sudo nano /etc/dnsmasq.conf
```

Add the following configuration options:

```
interface=wlan0
dhcp-range=192.168.4.2,192.168.4.20,255.255.255.0,24h
domain=wlan
address=/gw.wlan/192.168.4.1
```

Save your changes by pressing Ctrl + O followed by Ctrl + X.

**Create a Network Name and Password**

Configure your wireless access point by editing the hostapd configuration file.

To open this file for editing, run the following command:

```
sudo nano /etc/hostapd/hostapd.conf
```

Add some information about your wireless access point, including giving it a name and securing it with a password. To help protect your access point, your password should be eight characters or more and feature a mix of letters, numbers and symbols.

This tutorial creates an access point called **"ElevatorAir"** with the password **"password2020"** – make sure you use something more secure for your own network!

```
interface=wlan0
ssid=ElevatorAir
hw_mode=g
channel=7
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=password2020
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

Save your changes by pressing Ctrl + O followed by Ctrl + X.

## Setup additional packages

In ssh terminal, execute following commands.

```
sudo pip3 install Adafruit_DHT
sudo pip3 install grove
curl -sL https://github.com/Seeed-Studio/grove.py/raw/master/install.sh | sudo bash -s -
```

## Setup RTC service for Grove RTC module

- Setup additional packages

  ```
  sudo apt-get install python-smbus i2c-tools
  ```

- Setting up the Raspberry Pi RTC Time

  With I2C successfully setup and verified that we could see our RTC circuit then we can begin the process of configuring the Raspberry Pi to use our RTC Chip for its time.

  1. To do this, we will first have to modify the Raspberry Pi’s boot configuration file so that the correct Kernel driver for our RTC circuit will be successfully loaded in.

  Run the following command on your Raspberry PI to begin editing the /boot/config.txt file.

  ```
  sudo nano /boot/config.txt
  ```

  2. Within this file, you will want to add one of the following lines to the bottom of the file, make sure you use the correct one for the RTC Chip you are using. In our case, we are using a **DS1307**.

  ```
  dtoverlay=i2c-rtc,ds3231
  ```

  Once you have added the correct line for your device to the bottom of the file you can save and quit out of it by pressing Ctrl + X, then Y and then Enter.

  3. With that change made we need to restart the Raspberry Pi, so it loads in the latest configuration changes.

  Run the following command on your Raspberry Pi to restart it.

  ```
  sudo reboot
  ```

  4. Once your Raspberry Pi has finished restarting we can now run the following command, this is so we can make sure that the kernel drivers for the RTC Chip are loaded in.

  ```
  sudo i2cdetect -y 1
  ```

  You should see a wall of text appear, if UU appears instead of 68 then we have successfully loaded in the Kernel driver for our RTC circuit.

  5. Now that we have successfully got the kernel driver activated for the RTC Chip and we know it’s communicating with the Raspberry Pi, we need to remove the fake hwclock package. This package acts as a placeholder for the real hardware clock when you don’t have one.

  Type the following two commands into the terminal on your Raspberry Pi to remove the fake-hwclock package. We also remove hwclock from any startup scripts as we will no longer need this.

  ```
  sudo apt-get -y remove fake-hwclock
  sudo update-rc.d -f fake-hwclock remove
  sudo systemctl disable fake-hwclock
  ```

  6. Now that we have disabled the fake-hwclock package we can proceed with getting the original hardware clock script that is included in Raspbian up and running again by commenting out a section of code.

  Run the following command to begin editing the original RTC script.

  ```
  sudo nano /lib/udev/hwclock-set
  ```

  7. Find and comment out the following three lines by placing # in front of it as we have done below.

  Find

  ```
  if [ -e /run/systemd/system ] ; then
      exit 0
  fi
  ```

  Replace With

  ```
  #if [ -e /run/systemd/system ] ; then

  # exit 0

  #fi
  ```

  Once you have made the change, save the file by pressing Ctrl + X then Y then Enter.

- Set TimeZone

  Using **raspi-config** in terminal,

  _4 Localisation Options / I2 Change Time Zone / US / [Choose your Zone]_

- Syncing time from the Pi to the RTC module

  Now that we have our RTC module all hooked up and Raspbian and the Raspberry Pi configured correctly we need to synchronize the time with our RTC Module. The reason for this is that the time provided by a new RTC module will be incorrect.

  1. You can read the time directly from the RTC module by running the following command if you try it now you will notice it is currently way off our current real-time.

  ```
  sudo hwclock -D -r
  ```

  2. Now before we go ahead and sync the correct time from our Raspberry Pi to our RTC module, we need to run the following command to make sure the time on the Raspberry Pi is in fact correct. If the time is not right, make sure that you are connected to a Wi-Fi or Ethernet connection.

  ```
  date
  ```

  3. If the time displayed by the date command is correct, we can go ahead and run the following command on your Raspberry Pi. This command will write the time from the Raspberry Pi to the RTC Module.

  ```
  sudo hwclock -w
  ```

  4. Now if you read the time directly from the RTC module again, you will notice that it has been changed to the same time as what your Raspberry Pi was set at. You should never have to rerun the previous command if you keep a battery in your RTC module.

  ```
  sudo hwclock -r
  ```

- Setup syncing date time of Pi from RTC when bootup

  In terminal, execute

  ```
  sudo nano /etc/rc.local
  ```

  and add the following lines to the file:

  ```
  sudo hwclock -s
  date
  ```

  ![PiSSH](images/rclocal.png)

  To save the file, press Ctrl+X, Y then return.

## Setup systemd service for auto start

- Copy **elevator_ble** directory to /home/pi/ of Pi via SFTP.

- Create new service in ssh terminal,

  ```
  sudo nano /etc/systemd/system/elevatorair.service
  ```

  Copy following lines:

  ```
  [Unit]
  Description=Elevator Air Service for RPI Zero W

  [Service]
  TimeoutStartSec=0
  Type = simple
  Restart=always
  RestartSec=15s
  WorkingDirectory=/home/pi/elevator_ble
  ExecStart=/bin/bash start.sh

  [Install]
  WantedBy=multi-user.target
  ```

  To save the file, press Ctrl+X, Y then return.

  ```
  cd elevator_ble
  sudo chmod +x start.sh
  sudo systemctl enable elevatorair.service
  sudo systemctl start elevatorair.service
  sudo reboot
  ```

## Finding Pi Zero W via WiFi and connecting to admin web

Find WiFi hotspot within your mobile or WiFi builtin machine.

Hotspot ssid: `ElevatorAir`, password: `password2020`.

_We could change ssid and password easily within section **Create a Network Name and Password**_.

After connected to it, you could browse http://192.168.4.1

## BLE peripherals functionalties

- BLE Service & Charateristics

  Service: `0d7e0001-3e26-454e-9669-1a8b67b52161`

  1. Peripheral characteristic: `0d7e0002-b5a3-f393-e0a9-e50e24dcca9e`

     - _notify_

       _Every 4sec, it notifies current sensor variables etc._

       `fan:<on/off>, light:<on/off>, temp:<temperature of sensor>, humi:<humidity of sensor>, smoke:<ppm value>`

       ex: `fan:on, light:off, temp:27, humi:68, smoke:0`

  2. Configuration characteristic: `0d7e0003-b5a3-f393-e0a9-e50e24dcca9e`

     - _read_

       _You can read current full configuration._

       example:

       ```
       {
         "advertise_name": "Elevator_Air",
         "system_mode": true,
         "smoke_detection": true,
         "temperature_monitor": true,
         "temperature_threshold": 90
       }
       ```

       _advertise_name: BLE advertising name_

       _system_mode: If true, automatic control mode, false for manual override mode._

       _smoke_detection: Enable/Disable detection for Smoke alarm._

       _temperature_monitor: Enable/Disable to monitor temperature._

       _temperature_threshold: Threshold of monitoring temperature._

     - _write_

       _You could change configuration_

       example:

       ```
       {
         "temperature_threshold": 25
       }
       ```

       _You could add one or several fields in json message._

  3. Forcing relay characteristic: `0d7e0004-b5a3-f393-e0a9-e50e24dcca9e`

     - _write_

       _In manual override mode, you could force on/off for Fan and switch fast/slow speed for Fan._

       example:

       ```
       {
         "fan": "off",
         "speed": "fast",
       }
       ```

       _You could add only one of "fan" or "speed" field_

  4. Choose schedule characteristic: `0d7e0005-b5a3-f393-e0a9-e50e24dcca9e`

     - _read_

       _You can read current schedules._

       example:

       ```
         [
           {
             "id": "Morning",
             "start": "08:30",
             "end": "10:15",
             "dayofweeks": [0, 1, 2, 3, 4, 5, 6],
             "active": true,
             "fan": "on",
             "light": "on"
           },
           {
             "id": "Rush hour",
             "start": "11:30",
             "end": "14:00",
             "dayofweeks": [0, 1, 2, 3, 4, 5, 6],
             "active": true,
             "fan": "on",
             "light": "on"
           },
           {
             "id": "Overnight",
             "start": "00:00",
             "end": "06:30",
             "dayofweeks": [0, 1, 2, 3, 4, 5, 6],
             "active": true,
             "fan": "on",
             "light": "on"
           },
           {
             "id": "Evening",
             "start": "17:30",
             "end": "23:59",
             "dayofweeks": [0, 1, 2, 3, 4, 5, 6],
             "active": true,
             "fan": "on",
             "light": "on"
           }
         ]
       ```

     - _write_

       _You can update current schedules._

       example:

       ```
         [
           {
             "id": "Morning",
             "start": "08:30",
             "end": "10:15",
             "dayofweeks": [0, 1, 2, 3, 4, 5, 6],
             "active": true,
             "fan": "on",
             "light": "on"
           },
           {
             "id": "Rush hour",
             "start": "11:30",
             "end": "14:00",
             "dayofweeks": [0, 1, 2, 3, 4, 5, 6],
             "active": true,
             "fan": "on",
             "light": "on"
           },
           {
             "id": "Overnight",
             "start": "00:00",
             "end": "06:30",
             "dayofweeks": [0, 1, 2, 3, 4, 5, 6],
             "active": true,
             "fan": "on",
             "light": "on"
           },
           {
             "id": "Evening",
             "start": "17:30",
             "end": "23:59",
             "dayofweeks": [0, 1, 2, 3, 4, 5, 6],
             "active": true,
             "fan": "on",
             "light": "on"
           }
         ]
       ```

  5. System time characteristic: `0d7e0006-b5a3-f393-e0a9-e50e24dcca9e`

     - _notify_

       _Every 4sec, it will notify current system date & time of Pi Zero W._

       example:

       ```
       2020-08-08T09:16:02
       ```

     - _write_

       _You could change system date & time of Pi and RTC module._

       example:

       ```
       2020-08-08T09:16:02
       ```
