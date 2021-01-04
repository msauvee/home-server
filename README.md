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
* static ip on wlan0

The requirements are:
* a computer to flash the SD: we will use a mac
* the url of hypriot: we will use las version on on hypriot github https://github.com/hypriot/image-builder-rpi/releases/download/v1.12.3/hypriotos-rpi-v1.12.3.img.zip
* credention to the wifi: ssid and password (do not use quote in the `flash`command line)
* know the device id (SD card) on the computer (see bellow)
* check docker version did not changed (see [bellow](#find-the-device-id))

### Find the device id 
To know the device (SD card) on the computer, launch `diskutil list` on macos:
```
/dev/disk5 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *15.9 GB    disk5
   1:             Windows_FAT_32 NO NAME                 15.9 GB    disk5s1
```
We will use `/dev/disk5`. `flash` will ask to confirm the device.

### Docker version
The current docker version is `20.10.1, build 831ebea`. If docker is a newer version, it may have change the iptables. The current section on iptables of `/boot/user-data.yml` may be obsolete. You will see that if you can't connect to docker images or start docker. See [this section](#how-to-setup-iptables) if this is the case.

### Raspberry Zero or 3

If the raspberry model is Zero or 3, the UART should be desablez, so add `--bootconf ./boot/no-uart-config-for-pi0-3.txt`to the flash command line.
The file ./boot/user-data.yml will probably not work as it reply opn apt-get update/upgrade by cloud-init which require eth0. You can move apt-get update/upgrade in `runcmd` section.

Flash with the following command (replace <my_wifi_ssid> and <my_wifi_password>): 
`$ flash --hostname <hostname> --bootconf ./boot/no-uart-config-for-pi0-3.txt --userdata ./boot/user-data.yml --ssid <my_wifi_ssid> --password <my_wifi_password> https://github.com/hypriot/image-builder-rpi/releases/download/v1.12.3/hypriotos-rpi-v1.12.3.img.zip`


### Raspberry 4

The raspberry should be connected on ethernet as the apt-get update/upgrade are done before wlan0 is up !

Flash with the following command (replace <my_wifi_ssid> and <my_wifi_password>): 
```
$ flash --hostname <hostname> \
  --userdata ./boot/user-data.yml \
  --ssid <my_wifi_ssid> \
  --password <my_wifi_password> \
  https://github.com/hypriot/image-builder-rpi/releases/download/v1.12.3/hypriotos-rpi-v1.12.3.img.zip
```

Connect through ssh, and check the apt-get update and upgrade occurs correctly ((this bug)[https://github.com/hypriot/image-builder-rpi/issues/304] is not fixed adn still occures often). The command `$ apt-cache policy iptables-persistent` should return somthing like:
```
iptables-persistent:
  Installed: 1.0.11+deb10u1
  Candidate: 1.0.11+deb10u1
  Version table:
 *** 1.0.11+deb10u1 500
        500 http://raspbian.raspberrypi.org/raspbian buster/main armhf Packages
        100 /var/lib/dpkg/status
```
If you do not get the expected result, launch `$ sudo cloud-init clean --reboot`.

Check the SSD is mounted with the command `$ ls /opt/data` which should not be empty.

If you do not get the expected result, launch `$ sudo reboot`.

## How to install Nginx

Use image fropm https://hub.docker.com/r/tobi312/rpi-nginx/
* create conf files in /opt/data/homeserv0/nginx/conf
* create content files in /opt/data/homeserv0/nginx/wwww
* add default error pages

Get files from project under `nginx` folder and put them on the SSD under server folder.

Todo: 
* create an nginx image file that to apt-get/updgrade 

### Setup let's encrypt

Create a folder `letsencrypt/live` under `nginx`.
Create a folder `_letsencrypt`under `nginx/wwww`.
Generate Diffie-Hellman keys by running this command on your server: `openssl dhparam -out /opt/data/<hostname>/nginx/dhparam.pem 2048`
Comment out SSL related directives in the configuration: 
`$ sudo sed -i -r 's/(listen .*443)/\1;#/g; s/(ssl_(certificate|certificate_key|trusted_certificate) )/#;#\1/g' /opt/data/<hostname>/nginx/sites-available/sauvee.com.conf`

Start nginx image:
```
$ docker run --name nginx -d -p 80:80 -p 443:443 \
-v /opt/data/<hostname>/nginx/conf:/etc/nginx:ro \
-v /opt/data/<hostname>/nginx/www:/var/www \
-v /opt/data/<hostname>/nginx/letsencrypt:/var/letsencrypt \
arm32v7/nginx
```

Connect to the image in ssh: `$ docker exec -it nginx /bin/bash`
And install cerbot:
```
apt-get update
apt-get upgrade -y
apt-get install certbot -y
```


### How to setup iptables ?

If docker version change, the iptables enforced by docker may change. You have to redo the folowwing command to generate an iptables configuration taking into account docker rules and our required rules.

You have to flash without the content on `/ect/iptables/iptables.conf` of `/boot/user-data.yml`(you can comment it).
Log through ssh to the server and do the following instructions.
* create a file `iptable-filter.conf` in `/etc/iptables/` with the content of `/boot/iptable-filter.conf`.
* load it without flushing other tables: `$ sudo iptables-restore -n /etc/iptables/iptable-filter.conf`
* saves yout current iptables: `$ sudo iptables-save -f /etc/iptables/iptable.conf`
* update the section on `/ect/iptables/iptables.conf` of `/boot/user-data.yml`

Good to know:
 * to save current iptables con figuration into a file : `$ sudo iptables-save -f /etc/iptables/iptables.conf`
 * to load iptables from a file : `$ sudo iptables-restore /etc/iptables/iptables.conf`


## Useful sources

 * [Headless Raspberry Pi 4 SSH WiFi Setup (Mac + Windows, 10 Steps)](https://desertbot.io/blog/headless-raspberry-pi-4-ssh-wifi-setup)
 * [Hypriot](https://blog.hypriot.com/)
 * [Hypriot flash documentation](https://github.com/hypriot/flash)
 * [Cloud-init documentation](https://cloudinit.readthedocs.io/en/18.3/)
 * [How To Implement a Basic Firewall Template with Iptables on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-implement-a-basic-firewall-template-with-iptables-on-ubuntu-14-04)
 * [Deploying K8s on Raspberry Pi4 with Hypriot and Cloud-Init](http://www.jedimt.com/2019/12/deploying-k8s-on-raspberry-pi4-with-hypriot-and-cloud-init/)
 * [External storage configuration](https://www.raspberrypi.org/documentation/configuration/external-storage.md)
 * [How to get Docker running on your Raspberry Pi using Mac OS X](https://blog.hypriot.com/getting-started-with-docker-and-mac-on-the-raspberry-pi/)
 * [Deploying NGINX and NGINX Plus on Docker](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-docker/)
 * [Docker and iptables](https://docs.docker.com/network/iptables/)
 * [Sécuriser mon serveur Docker avec un pare-feu minimaliste](https://www.grottedubarbu.fr/docker-firewall/)
 * [inrupt](https://inrupt.com/) 