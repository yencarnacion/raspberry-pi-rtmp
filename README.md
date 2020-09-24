Output rtmp stream to HDMI using a raspberry pi 4
==================================================

## I used the following hardware
Vilros Raspberry Pi 4 Complete Starter Kit
with Fan-Cooled Heavy-Duty Aluminum Alloy Case
(4GB RAM, Black Case)
https://www.amazon.com/gp/product/B07XTQL6YZ/

## Adapted from
How to make a DIY Streaming Bridge
with a Raspberry Pi for the ATEM Mini and OBS
https://www.youtube.com/watch?v=MtETl23cnOA

## Steps
* Install Raspberry Pi OS Lite from https://www.raspberrypi.org/downloads/
    * default username: pi password: raspberry (you should change the password)
* enable wifi
    * choose country
        * sudo raspi-config
            * under Localization Options
            * set wifi under credentials under Network Options
* enable SSH
    * sudo raspi-config
        * under Interfacing Options
* change hostname
    * sudo raspi-config
        * Under Network Options
            * if you use hdmi as hostname you can access as hdmi.lan or hdmi.local from the local network

* Video configuration
    * all raspberry pi hdmi options
        * https://www.raspberrypi.org/documentation/configuration/config-txt/video.md
    * edit /boot/config.txt
        * #disable_overscan=1 # comment out
        * hdmi_force_hotplug=1
        * hdmi_drive=2
        * hdmi_group=1
        * hdmi_mode=16 # force to 1080p @60Hz
    * edit /boot/cmdline.txt
        * disable console blanking kernel option
            * add consoleblank=0
            * mine looks as follows
            ```
            console=serial0,115200 console=tty1 root=PARTUUID=91fca638-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait consoleblank=0
            ```

* sudo apt-get install update
* sudo apt-get install nginx libnginx-mod-rtmp -y

* sudo usermod -aG video www-data

* edit /etc/nginx/nginx.conf
    * add what follows to bottom of file
    ```
    rtmp {
         server {
              listen  1935;
              application live {
                  live on;
                  record off;
                  allow play 127.0.0.1;
                  deny play all;
                  exec omxplayer -o hdmi rtmp://127.0.0.1/live/$name;
              }
         }
    }
    ```
    * check config with $ sudo nginx -t
    * restart with $ sudo nginx -s reload

* sudo apt-get install omxplayer -y

* sudo shutdown -r now

```
You can now stream to
rtmp://<pi-hostname>.lan/live/hdmi
or
rtmp://<pi-hostname>.local/live/hdmi
or
rtmp://<pi-ip-address>/live/hdmi
```


## if you have problems, info below can help

The following command outputs the current HDMI status,
including mode and resolution
```
$ tvservice -s
```

Also useful

https://www.raspberrypi.org/documentation/configuration/hdmi-config.md

and

https://www.opentechguides.com/how-to/article/raspberry-pi/28/raspi-display-setting.html


Run the tvservice command to output the result to a file.
```
$ tvservice -d edid.dat
```

Pipe the file to edidparser to generate a readable text file.
```
$ edidparser edid.dat > edid.txt
```

## OBS setup for live steaming


Look at

https://www.youtube.com/watch?v=pBQWUnKzQtA

and

https://www.youtube.com/watch?v=yTt5IxFWaEM
