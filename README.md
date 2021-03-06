# Home Server
a home server for my Raspberry Pi 3

## Setup
### Ubuntu 20.10 Server
```
sudo apt update
sudo apt upgrade
```

### NginX

```
sudo apt install nginx
sudo ufw app list
sudo ufw status
systemctl status nginx
```
### Lets Encrypt-Certificats
call http://raspberrypi call http://domain
```
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
sudo certbot renew --dry-run
```
### NginX configuration
```
cd /etc/nginx/sites-available
$ sudo nano default
sudo systemctl stop nginx
sudo systemctl start nginx
sudo systemctl restart nginx
```
If you want to reload configuration:
```
sudo systemctl reload nginx
```
### External USB disk

Get the UUID of the disk:

```
sudo blkid
```

Enter this in 

```
sudo nano /etc/fstab
```

```
UUID=04F20EEDF20EE332   /media/video    ext4    defaults,nofail 0       1
```

Then enter

```
sudo mkdir /media/video
```

mount drive:

```
sudo mount -a
```

```
cd /media
ll
sudo chmod 777 /media/video
```

Send external disk to sleep after 10 min:

Install ```tlp```:

```
sudo apt install tlp
```

Get disk ID for tlp:

```
sudo tlp diskid
```

```
sudo nano /etc/tlp.conf
```
```
# Disk devices; separate multiple devices with spaces (default: sda).
# Devices can be specified by disk ID also (lookup with: tlp diskid).
DISK_DEVICES="ata-TOSHIBA_MQ01ABD100_238MSIE4S"

# Disk advanced power management level: 1..254, 255 (max saving, min, off).
# Levels 1..127 may spin down the disk; 255 allowable on most drives.
# Separate values for multiple disks with spaces. Use the special value 'keep'
# to keep the hardware default for the particular disk.
DISK_APM_LEVEL_ON_AC="127"
DISK_APM_LEVEL_ON_BAT="127"

# Hard disk spin down timeout:
#   0:        spin down disabled
#   1..240:   timeouts from 5s to 20min (in units of 5s)
#   241..251: timeouts from 30min to 5.5 hours (in units of 30min)
# See 'man hdparm' for details.
# Separate values for multiple disks with spaces. Use the special value 'keep'
# to keep the hardware default for the particular disk.
DISK_SPINDOWN_TIMEOUT_ON_AC="120"
DISK_SPINDOWN_TIMEOUT_ON_BAT="120"
```
```
sudo systemctl enable tlp
sudo systemctl start tlp
```

## Install Home Server
### Node.js
Update your system package list:

```
pi@w3demopi:~ $ sudo apt-get update
```
Upgrade all your installed packages to their latest version:

```
pi@w3demopi:~ $ sudo apt-get dist-upgrade
```

To download and install newest version of Node.js, use the following command:
```
pi@w3demopi:~ $ curl -sL https://deb.nodesource.com/setup_15.x | sudo -E bash -
```

Now install it by running:

```
pi@w3demopi:~ $ sudo apt-get install -y nodejs
```

### Compile on raspberry

Copy all files of this project tothe target directory on the raspberry.

```
npm i in directory HomeServer
export MEDIA_PATH=/media/video/videos
node .
```

Run by 

```
http://raspberrypi:9865<path>...
```

### Install as service

```
sudo nano /lib/systemd/system/home_server.service
```

```
[Unit]
Description=Home Server for serving videos to AMAZON FirePlayer
Documentation=https://github.com/uriegel/HomeServer/blob/master/README.md
After=network.target

[Service]
Environment=VIDEO_PATH=/media/videos
Environment=MUSIC_PATH=/media/music
Environment=RELATIVE_URL=/media
Type=simple
User=uwe
ExecStart=/usr/bin/node /home/uwe/HomeServer
Restart=on-failure

[Install]
WantedBy=multi-user.target

```

```
sudo systemctl daemon-reload
sudo systemctl enable home_server.service
sudo systemctl start home_server
```


// TODO: /etc/nginx/sites-available/default:
// Items redirecting to nodejs
// Install node.js as service

```
location <path>; {
	proxy_pass http://localhost:9865/<path>;
}

location / {
```