# Home Server

## How to setup home server

Targeted service:
* personal docker registry service on internet
* personal [inrupt](https://inrupt.com/) service on internet
* [personal home page](http://msauvee.com) on internet
* other familly member home page on internet

List of elements to setup
* setup device to flash raspberry
* network
* server hardware
  * raspebrry pi 4 B 4Gb
  * SSD hard drive
* server software
  * OS : [Hypriot](https://blog.hypriot.com/) for docker
  * docker
  * docker registry
  * nginx
  * backup 

## How to setup device to flash raspberry

Use [Hypriot documentation](https://github.com/hypriot/flash).
For macos:
```
curl -LO https://github.com/hypriot/flash/releases/download/2.7.0/flash
chmod +x flash
sudo mv flash /usr/local/bin/flash
brew install pv
brew install awscli
```

## How to install Hypriot on raspberry

The goal of this step is to flash the server, which contains the following elements:
* change host name (black-pearl would provide too much info) to home-srv0
* set timezone, add ntp szervice
* no root, only ssh for msauvee
* add wifi setup

The requirements are:
* a computer to flash the SD: we will use a mac
* the url of hypriot: we will use las version on on hypriot github https://github.com/hypriot/image-builder-rpi/releases/download/v1.12.3/hypriotos-rpi-v1.12.3.img.zip
* credention to the wifi: ssid and password (do not use quote in the `flash`command line)
* knwon the device (SD card) on the computer

To know the device (SD card) on the computer, launch `diskutil list` on macos:
```
/dev/disk5 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *15.9 GB    disk5
   1:             Windows_FAT_32 NO NAME                 15.9 GB    disk5s1
```
We will use `/dev/disk5`. `flash` will ask to confirm the device.

### Raspberry Zero or 3

If the raspberry model is Zero or 3, the UART should be desablez, so add `--bootconf ./boot/no-uart-config-for-pi0-3.txt`to the flash command line.

Flash with the following command (replace <my_wifi_ssid> and <my_wifi_password>): 
`$ flash --hostname <hostname> --bootconf ./boot/no-uart-config-for-pi0-3.txt --userdata ./boot/user-data.yml --ssid <my_wifi_ssid> --password <my_wifi_password> https://github.com/hypriot/image-builder-rpi/releases/download/v1.12.3/hypriotos-rpi-v1.12.3.img.zip`

### Raspberry 4

Flash with the following command (replace <my_wifi_ssid> and <my_wifi_password>): 
`$ flash --hostname <hostname> --userdata ./boot/user-data.yml --ssid <my_wifi_ssid> --password <my_wifi_password> https://github.com/hypriot/image-builder-rpi/releases/download/v1.12.3/hypriotos-rpi-v1.12.3.img.zip`

## How to install docker

see [this tutorial](https://blog.hypriot.com/getting-started-with-docker-and-mac-on-the-raspberry-pi/)

## How to install Nginx

Use image fropm https://hub.docker.com/r/tobi312/rpi-nginx/
Create conf files in tata/nginx
`$ mkdir -p /home/msauvee/data/nginx/{.ssl,html} && mkdir -p /home/msauvee/data/nginx/.config/nginx && touch /home/msauvee/data/nginx/.config/nginx/default.conf`

`$ docker run --name nginx -d -p 80:80 -p 443:443 -v /home/msauvee/data/nginx/.ssl:/etc/nginx/ssl:ro -v /home/msauvee/data/nginx/.config/nginx:/etc/nginx/conf.d:ro -v /home/msauvee/data/nginx/html:/var/www/html tobi312/rpi-nginx`


## Useful sources

 * [Headless Raspberry Pi 4 SSH WiFi Setup (Mac + Windows, 10 Steps)](https://desertbot.io/blog/headless-raspberry-pi-4-ssh-wifi-setup)
 * [Hypriot](https://blog.hypriot.com/)
 * [Hypriot flash documentation](https://github.com/hypriot/flash)
 * [Clou-init documentation](https://cloudinit.readthedocs.io/en/18.3/)
 * [inrupt](https://inrupt.com/) 