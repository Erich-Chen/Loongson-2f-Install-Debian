# Loongson-2f-Install-Debian
My experience to install Debian on loongson-2f  
ref. https://wiki.debian.org/DebianYeeloong/HowTo/Install  

## Hardware
I bought a Lemote YeeLoong 8089D with CNY279 at Oct. 23rd, year 2013, 4 years ago "yesterday".  

## Netboot Files
Download netboot files:  
Wheezy (Debian 7.11) : http://ftp.debian.org/debian/dists/wheezy/main/installer-mipsel/current/images/loongson-2f/netboot/  
or
Jessie (Debian 8.9)  : http://ftp.debian.org/debian/dists/jessie/main/installer-mipsel/current/images/loongson-2f/netboot/  
I choose Jessie :-)  

* Unfortunately Debian dropped images for loongson-2f. https://lists.debian.org/debian-devel-announce/2016/07/msg00000.html  

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
```
su -                 # switch to root to enable sudo for current user
apt install sudo
echo '${USER}  ALL=(ALL:ALL) ALL' >> /etc/sudoers
exit                 # switch back to current user

# Install NTP and setup timezone
sudo apt install ntp
echo "Asia/Shanghai" > /etc/timezone && dpkg-reconfigure -f noninteractive tzdata
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
```
cd /tmp
wget https://github.com/biergaizi/loongson-sources/releases/download/v3.16.4/linux-3.16.4-yeeloong-gaizi.tar.xz 
tar xf linux-3.16.4-yeeloong-gaizi.tar.xz
sudo cp -r 3.16.4-yeeloong-gaizi/boot/* /boot
sudo cp -r 3.16.4-yeeloong-gaizi/lib/* /lib
sudo update-grub
sudo reboot
```
