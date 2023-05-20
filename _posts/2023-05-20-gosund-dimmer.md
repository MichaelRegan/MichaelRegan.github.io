---
title: Gosund Dimmer
date: 2023-05-21 11:35:00
categories: [Projects, IOT]
tags: [lights, tasmota]
---
# Configure Gosund Dimmer with Tasmota
Plug in the module
Start the flash tool on the raspberry PI (MRPiUtil)
```sh
ssh michael@MRPiUtil.mjsquared.net
```
## Flash the module
* reconnect wifi to the tasmoto wifi
* Update wifi settings
* connect on the AVNet wifi
* Update firmware to custom from google drive
* Restore backup configuration from google drive
* Configure name in wifi
* Configure friendly name in other
* Issue SetOption31 1 to stop the green light blinking
* Add static IP after 192.168.20.149

## Update settings for MQTT
Add server: 123.123.123.123
Add tasmota/devicename for topic
SetOption19 1 to set Home Assistant discovery

## HA
Click on integrations
Add new device to lovelace
https://www.youtube.com/watch?v=mWQnHResSCM

notes:

    GPIO0 = button
    GPIO1 = green led
    GPIO14 = relay
    GPIO16 = red led
