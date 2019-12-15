# Loongson-2f-Install-Debian
My experience to install Debian on loongson-2f  
ref. https://wiki.debian.org/DebianYeeloong/HowTo/Install  

## Hardware
I bought a Lemote YeeLoong 8089D with CNY279 at Oct. 23rd, year 2013.  

## Netboot Files
Download netboot files:  
Wheezy (Debian 7.11) : http://ftp.debian.org/debian/dists/wheezy/main/installer-mipsel/current/images/loongson-2f/netboot/  
http://archive.debian.org/debian-archive/debian/dists/wheezy/main/installer-mipsel/current/images/loongson-2f/netboot/
You will find the following files:
```
Name                          Last modified       Size
boot.cfg                      2016-05-30 23:04    129
initrd.gz                     2016-05-30 23:04    5.0M
vmlinux-3.2.0-4-loongson-2f   2016-05-30 23:04    7.9M
```

or
Jessie (Debian 8.9)  : ~~http://ftp.debian.org/debian/dists/jessie/main/installer-mipsel/current/images/loongson-2f/netboot/~~  
I choose Jessie :-)  
UPDATE: Now moved to : http://archive.debian.org/debian-archive/debian/dists/jessie/main/installer-mipsel/current/images/loongson-2f/netboot/

I have downloaded the following files:
```
Name	                           Last modified	       Size
boot.cfg                          2018-06-19 09:20      130
initrd.gz                         2018-06-19 09:20      12M
vmlinux-3.16.0-6-loongson-2f      2018-06-19 09:20      11M
```

**NOTE** Unfortunately Debian dropped images for loongson-2f. https://lists.debian.org/debian-devel-announce/2016/07/msg00000.html 

## TFTP Server
Run tftpd32 (http://tftpd32.jounin.net/) on my windows 10 computer, and point 'current folder' to where netboot files saves. 
Connect YeeLoong with ehternet cable into the same LAN with TFTP server.  

By the way, a fast and stable internet is also very important, especially if you are in China.  

## Boot from TFTP server
Press DEL (repeatedly) on startup to enter PMON (the Yeeloong's BIOS);  
Delete any extra characters;  
Type:  
```
PMON> ifaddr rtl0 192.168.0.bb
PMON> load tftp://192.168.0.aa/vmlinux-3.16.0-4-loongson-2f
PMON> initrd tftp://192.168.0.aa/initrd.gz
PMON> g
```
Here 192.168.0.bb is IP address of YeeLoong laptop. Mind the letter 'l' and numerical '0' in 'rtl0'.  
192.168.0.aa is the IP of TFTP server (my win 10 laptop).  

## Installation 
*It is safe to close TFTP server after system boot, though of no harm to keep it open.*  
Not much special during system installation.  

I partited the harddrive (8.2 GiB) into three partitions. /boot must be ext2.    
```
/boot  ext2  100M
/      ext4  7G
       swap  1.1G
```

I selected Debian Desktop Enviroment and LXDE at TaskSel but I will disable GUI after installation. It is also a good choice to not install X-Desktop.  


## Post Installation  
Unable to start X-Desktop at first boot. Remove xserver-xorg-video-siliconmotion can *currently* solve the problem.  
Press ALT+F1 to switch to text mode / shell.  

```
su -                 # switch to root to enable sudo for current user
apt install sudo
echo '${USER}  ALL=(ALL:ALL) ALL' >> /etc/sudoers
exit                 # switch back to current user

# Remove Siliconmotion to fix GUI failure
sudo apt purge xserver-xorg-video-siliconmotion
```

## More setup
```
# Install NTP and setup timezone
sudo apt install ntp
echo "Asia/Shanghai" > /etc/timezone && dpkg-reconfigure -f noninteractive tzdata
# change "Asia/Shanghai" into your own timezone unless you are also in China. 
date

# Some software
sudo apt install vim mc byobu

# Default boot into text mode 
sudo systemctl set-default multi-user.target

# Start X-Desktop from shell
sudo systemctl start ligthdm

# Log out X-Desktop and free up memory
sudo systemctl stop lightdm
sync && echo 3 | sudo tee /proc/sys/vm/drop_caches
```

## Patch a new kernel 
https://github.com/biergaizi/loongson-sources  
DO NOT patch this kernel to Debian Wheezy.  
```
cd /tmp
wget https://github.com/biergaizi/loongson-sources/releases/download/v3.16.4/linux-3.16.4-yeeloong-gaizi.tar.xz 
tar xf linux-3.16.4-yeeloong-gaizi.tar.xz
sudo cp -r 3.16.4-yeeloong-gaizi/boot/* /boot
sudo cp -r 3.16.4-yeeloong-gaizi/lib/* /lib
sudo update-grub
sudo reboot
```

## Cooling down fan speed
ref. https://romanrm.net/loongson/yeeloong-fan  
```
sudo apt install lm-sensors 
sensors
sudo apt install thinkfan
sudo systemctl stop thinkfan.service
sudo cp /etc/thinkfan.conf{,.bak}
cat << EOL | sudo tee /etc/thinkfan.conf
sensor /sys/devices/virtual/hwmon/hwmon0/temp1_input
fan /sys/class/hwmon/hwmon0/pwm1
(0, 0, 62)
(1, 60, 64)
(2, 62, 66)
(3, 64, 32767)
# The format of the threshold lines is (fan step, temperature on which to step down, temperature on which to step up).
EOL
sudo systemctl start thinkfan.service
```
The fan will almost always quite.  

## Debian Wheezy (7.x) X-Desktop fix
```
sudo apt remove xserver-xorg-video-siliconmotion
wget --no-check-certificate https://www.anheng.com.cn/loongson2f/wheezy/xserver-xorg-video-siliconmotion/xserver-xorg-video-siliconmotion_1.7.6-1loongson_mipsel.deb
sudo dpkg -i xserver-xorg-video-siliconmotion_1.7.6-1loongson_mipsel.deb

cat << EOL | sudo tee /etc/X11/xorg.conf
# xorg.conf (X.Org X Window System server configuration file)  
#  
# This file was generated by dexconf, the Debian X Configuration tool, using  
# values from the debconf database.  
#  
# Edit this file with caution, and see the xorg.conf manual page.  
# (Type "man xorg.conf" at the shell prompt.)  
#  
# This file is automatically updated on xserver-xorg package upgrades *only*  
# if it has not been modified since the last upgrade of the xserver-xorg  
# package.  
#  
# If you have edited this file but would like it to be automatically updated  
# again, run the following command:  
#   sudo dpkg-reconfigure -phigh xserver-xorg  
  
Section "Device"  
        Identifier      "Card0"  
        Driver          "siliconmotion"  
        Option          "pci_burst" "true"  
        Option          "HWCursor" "true"  
        Option          "VideoKey" "45000"  
        Option          "UseBIOS" "false"  
        Option          "PanelSize" "1024x600"  
        Option          "CSCVideo" "false"  
EndSection  
  
Section "Screen"  
        Identifier      "Screen0"  
        Device          "Card0"  
        Monitor         "Monitor0"  
        DefaultDepth    16  
EndSection  
EOF

```
