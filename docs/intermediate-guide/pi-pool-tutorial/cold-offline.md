# Cold server setup

This can be any 64 bit capable Raspi. Any Raspberry Pi 4, Raspberry Pi 3b+ or a Raspberry Pi-400. I would use a Pi-400. Keyboard is built in which is convienant and safer. The cold machine is only used to sign transactions and is left powered down for 99% of the time(if not more). The Pi-400 can just be powered off and in a safe. Also unlike an online node, the Cold machine can be run from the sdcard. Just make sure you have mulitple copies of your keys just incase you get a bad sdcard. Or better yet clone this system onto 3 or 4 other bootable sdcards. Remember you can also boot from USB.

### Ubuntu or Raspbian

Raspberry Pi OS is a lot faster than the gnome desktop. There is now a 64bit image you can install but it is not available in rapi-imager selection, IDK why. Check out the images in the link below grab the latest version. It is a zip file so we have to unzip it.

[https://downloads.raspberrypi.org/raspios_arm64/](https://downloads.raspberrypi.org/raspios_arm64/)

Unzip the img file and flash it with Raspi-imager.

# Log in & setup user

Log in and open a terminal create the $USER and add it to sudoers. ada in our case. We have to create a new user so the uid, gid and name match. It is far less error prone than trying to change the user id of the defualt user. With the user/group id of 1001 you will not run into issues with permissions transfering between systems. You can delete the Pi/Ubuntu user after you log back in or leave it.

```bash
sudo adduser ada; sudo adduser ada sudo
```
log out and back in as $USER.

Disable the radios.

```bash
sudo rfkill block wifi
sudo rfkill block bluetooth
```

## USB transfer

Basically repeating the steps to setup an fstab entry. This is to mount the USB transfer disk at boot should you have it inserted when you power on. It also makes the mount command simpler. You can just sudo mount usb-transfer. You could also then make an alias for auto complete: usb-transfer. 

I would recommend against using usbmount for mounting on insert. It doesn't work well. Just manually mount it.


Create the mountpoint & set default ACL for files and folders with umask.

```bash
cd; mkdir $HOME/usb-transfer; umask 022 $HOME/usb-transfer
```

Attach the external drive and list all drives with fdisk.

```bash
sudo fdisk -l
```

Example output:

```bash
Disk /dev/sdb: 57.66 GiB, 61907927040 bytes, 120913920 sectors
Disk model: Cruzer          
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 75B71A3E-9E1A-4659-94D0-E0C949A26740

Device     Start       End   Sectors  Size Type
/dev/sdb1   2048 120913886 120911839 57.7G Linux filesystem
```

In my case it is /dev/sdb with one partition designated /dev/sdb1. Yours may be /dev/sdc, /dev/sdd or so on. /dev/sda is usually the system drive. If you are using an sdcard they are designated /dev/mmcblk0 to boot so inserting a USB stick into a system booting from sdcard can designate /dev/sda. Just be careful you are dealing with the correct drive.

Locate your drive and get the UUID for the partition we created earlier.

Run blkid and pipe it through awk to get the UUID of the filesystem we just created.

```bash
sudo blkid /dev/sdb1 | awk -F'"' '{print $2}'
```

Example output:

```bash
c2a8f8c7-3e7a-40f2-8dac-c2b16ab07f37
```

For me the UUID=c2a8f8c7-3e7a-40f2-8dac-c2b16ab07f37

Add a mount entry to the bottom of fstab adding your UUID and the full system path to you backup folder.

```bash
sudo nano /etc/fstab
```

```bash
UUID=c2a8f8c7-3e7a-40f2-8dac-c2b16ab07f37 /home/ada/usb-transfer auto nosuid,nodev,nofail 0 1
```

> nofail allows the server to boot if the drive is not inserted.

### Test your drive mounts

Mount the drive and confirm it mounted by locating listing the contents. You should see the ada folder, offline-transfer and lost&found. If they are not present then your drive is not mounted.

```bash
sudo mount usb-transfer; ls $HOME/usb-transfer
```
# System prep

The cold machine will never be online. We do not need the monitoring, cardano-node, nginx or a handful of the variables we created. It is also important to note that these machines do not have any cmos battery to keep time. It should not pose too many issues if any. Just be aware that when you fire up the cold machine it's time is off.

## Transfer contents of the USB stick.

```bash
cd; rsync -aP usb-transfer/ada/ ~/
```

Source the .adaenv file on login.

```bash
echo . ~/.adaenv >> ${HOME}/.bashrc
```

Switch the Stake Pool Operator scripts to 'offline mode'.

```bash
cd
sed -i stakepoolscripts/bin/common.inc \
    -e 's#offlineMode="no"#offlineMode="yes"#' 
```
Confirm.

```bash
. .adaenv; 00_common.sh
```

The wireless is disabled already. What I do is use a network jack without a cable inserted into the port to block it. This is to prevent a cable ever accidently being plugged into it. Or fill it with bubble gum..

## Key creation













