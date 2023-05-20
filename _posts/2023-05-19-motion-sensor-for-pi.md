---
title: PIR Motion Sensor for Pi
date: 2023-05-19 12:15:00
categories: [Projects, IOT]
tags: [hardware, raspberrypi, wall display]
---


# mrpir
Python script to support a PIR sensor on SBC's and publish over MQTT with Home Assistant discovery support

The following environment variables are used and should be stored in an ".env" file:

    MQTT_USER_NAME=xyz
    MQTT_PASSWORD=xyz
    MQTT_DEVICE=xyz
    MQTT_CLIENT_ID=xyz
    MQTT_PORT=1883 or other if configured differently
    MQTT_BROKER=servername or IP of the MQTT broker
    PIR_PIN=signal pin connected to the presense sensor
    MQTT_LOGGING=0 or 1 
    XSCREENSAVER_SUPPORT=1
    LOGGING_LEVEL=1


make sure python dependencies are installed:

    sudo pip3 install PyYAML paho-mqtt python-decouple

<br>

# Configure a raspberry pi as a KIOSK for Home Assistant
## Create a startup script
1. Edit your crontab list by typing:
    ``` sh
    sudo crontab -e
    ```
    You can launch crontab without entering sudo, but if you do, you won’t be able to run scripts that require admin privileges. In fact, you get a different list of crontabs if you don’t use sudo so don’t forget to keep using it or not using it.
2. Add a line at the end of the file that reads like this:
    ``` sh
    @reboot python3 /home/pi/startup.sh
    ```
3. Create the file: /home/pi/startup.sh
4. Run 
    ``` sh
    nano startup.sh
    ```
5. Add two lines: 
    ``` sh
    echo "${usb_flag}" | sudo tee /sys/devices/platform/soc/3f980000.usb/buspower >/dev/null
    sudo tvservice --off
    ```
6. make the startup file executable: sudo chmod a+x startup.sh
## Remove the cursor on the PI screen
``` sh
sudo sed -i -- "s/#xserver-command=X/xserver-command=X -nocursor/" /etc/lightdm/lightdm.conf
```
also see: [how to permanently hide mouse pointer or cursor on raspberry pi](https://raspberrypi.stackexchange.com/questions/53127/how-to-permanently-hide-mouse-pointer-or-cursor-on-raspberry-pi/53813#53813)

Or go the app route
``` sh
sudo apt install xdotool unclutter
```
## Automatically enter the kiosk mode and launch homeassistant url
1. Create a file called mrscreen.desktop (or something else .desktop) in the /etc/xdg/autostart/ directory.
``` bash
sudo nano /etc/xdg/autostart/mrscreen1.desktop
```
2. Use the following layout in the myapp.desktop file. 
``` toml
[Desktop Entry]
Exec=chromium-browser --noerrdialogs --disable-infobars --ignore-certificate-errors --kiosk https://homeassistant.mjsquared.net
```

## Add xscreensaver
``` sh
sudo apt-get update
apt-cache search xscreensaver*
sudo apt-get install xscreensaver*
```
Run from nvc or on machine, not ssh
``` sh
xhost +local:pi (this will let the mrpir services interact with the screensaver)
```
Test xscreensarver
``` sh
xscreensaver-command -display ":0.0" -activate
xscreensaver-command -display ":0.0" -deactivate
```
## Start mrpir:
``` sh
sudo nano /lib/systemd/system/mrpir.service
```
Past in the followign:
``` toml
[Unit]
Description=Presense sensor
After=multi-user.target
Wants=network-online.target systemd-networkd-wait-online.service

StartLimitIntervalSec=500
StartLimitBurst=15

[Service]
Type=idle
Restart=on-failure
RestartSec=5s

ExecStart=/usr/bin/sudo /usr/bin/python3 /home/pi/mrpir/mrpir.py (Or path to where you installed the service)
User=pi (or another username)

[Install]
WantedBy=multi-user.target
```
Configure the service file with the right permissions
``` sh
sudo chmod 644 /lib/systemd/system/mrpir.service
```

Link to the right direcotry
``` sh
ln -s “$(pwd)/mrpir.service” /lib/systemd/system/mrpir.service
```
Reload and enable the service
``` sh
sudo systemctl daemon-reload
sudo systemctl enable mrpir.service
```
Reboot just to ensure we are all set
```sudo reboot```

## Checking the log
``` sh
journalctl -u mrpir.service -n 100
```

### Exit kiosk mode from ssh
``` sh
pkill chromium
```
