# Software

## Operating System

Ubuntu Server 22.04 (64-bit)

## Bios

  

## Initial setup: BIOS and Ubuntu Installation on SD Card

What you will need:  

1- A computer with the ISO image of Ubuntu 22.04 downloaded.  
2- A USB drive with 4GB or more space to "burn" this ISO image from your computer. Note: this device will be erased and overwritten with the ISO image.   
3- Software in your computer to "burn" the ISO image to the USB disk; this depends on your O/S. [Here's one example of using dd with MacOS](https://www.raspberrypi.org/documentation/installation/installing-images/mac.md).  
4- A blank micro-SD card with 64GB or more space.
5- An Ethernet cable connectes to the Internet  

Detailed steps:  

1- [Download the Ubuntu Server 22.04 image to your computer.](https://ubuntu.com/download/server)  
2- Write the ISO image to the USB drive on your computer.  
3- Once the ISO image write is complete, remove the USB drive from your computer, and connect it to one of the USB ports of the fitlet2.  
4- Make sure the fitlet2 is powered off. Insert the blank SD card into the gateway.  Note: if you are using a SanDisk device, the colored/labeled side of the micro SD card should be facing the bottom of the fitlet2; make sure you insert until it latches.  
5- Connect USB mouse and keyboard to the fitlet2, and a cable to connect its HDMI output to a monitor, and the Ethernet cable.  
6- Power up the gateway, and press ESC on the keyboard during the fitlet2 splash screen to bring the BIOS menu.  
7- Under Main, use the down arrows to go to OS Selection. Hit Enter and select Linux.  
8- Use the right arrow to go to Boot menu.  
9- Change "Setup Prompt Timeout" to 1.  
10- Use up/down arrows to navigate to "Fixed Boot Order Priorities" and click Enter to select boot order #1 to #6 as follows.  
Boot Option #1 USB HDD  
Boot Option #2 USB CD/DVD  
Boot Option #3 USB Key  
Boot Option #4 SD  
Boot Option #5 HDD  
Boot Option #6 Network.  
11- Use right arrow to select "Save & Exit".  
12- You should see a GRUB window; use the down arrow to select "Try or Install Ubuntu Server".  
13- After some initialization, the Ubuntu installation user interface will show up. If it prompts to update the installer, select the default (no). Select English, English (US) keyboard; Done.  
14- Use the up arrows to select "Ubuntu Server (minimized)", Done.  
15- The eno1 or enp1s0 Ethernet network interface should  be listed as an interface this server can use. Check that the information is corect. Click on Done.  
16- The next screen asks for a proxy address; skip that by pressing Done.  
17- Also press Done for mirror address.  
18- Leave "Use an entire disk" selected, and leave the local disk selected (unless it shows a device with name FLEXXON_M2 as the local disk - in which case use the up arrow to verify the list of disks and make sure there is a local disk matching the size of the SD card). Make sure you select the SD card as local disk - do not install on the FLEXXON_M2! Click Done.  
20- Int he next screen you should see two devices: ubuntu-vg (new) and the FLEXXON_M2. Verify this is correct, click Done.  
21- In the "Confirm destructive action", down arrow then Continue.  
22- Enter server name, user name and password; **Note** make sure you store username and password in a password manager in your computer, so you can retrieve it later; Continue.  
23- Click Continue to "skip for now" upgrade to Ubuntu Pro.  
24- Click on "Install OpenSSH Server", navigate down to Done.  
25- In "Featured Server Snaps", skip and navigate down to Done.  
26- Now wait... After the installation is complete, power off the fitlet2.   
27- Remove the USB drive, but keep the SD card in the fitlet2.  
28- Power up the fitlet2 - it should now boot from the SD card.  

### Bios Update

**Optional - when in doubt, do not try this!** if you need to update the BIOS, information is available [here](http://www.fit-pc.com/wiki/index.php?title=Fitlet2:_BIOS_Update).

## Upgrade Ubuntu

When asked for which services to restart, enter 11 for none of the above

```
sudo apt update -y
sudo apt upgrade -y
sudo apt autoremove -y
```

## Install Vim

```
sudo apt install -y vim
```

## Change Hostname

Edit the hostname in the following files:

```
sudo vi /etc/hostname
sudo vi /etc/hosts
```

Change the hostname without a reboot:

```
sudo hostname <new_hostname>
```

## Disable Auto Updates and Unattended Upgrades

Edit the following file:

```
sudo vi /etc/apt/apt.conf.d/20auto-upgrades
```

The file should look like as follows:

```
APT::Periodic::Update-Package-Lists "0";
APT::Periodic::Unattended-Upgrade "0";
```

## Recover from Kernel Panics and Lockups

Create the following file:

```
sudo vi /etc/sysctl.d/panic.conf
```

Add the following lines to that:

```
# Reboot this many seconds after panic
kernel.panic = 20

# Panic if the kernel detects an I/O channel
# check (IOCHK). 0=no | 1=yes
kernel.panic_on_io_nmi = 1

# Panic if a hung task was found. 0=no, 1=yes
kernel.hung_task_panic = 1

# Setup timeout for hung task,
# in seconds (suggested 300)
kernel.hung_task_timeout_secs = 300

# Panic on out of memory.
# 0=no | 1=usually | 2=always
vm.panic_on_oom=2

# Panic when the kernel detects an NMI
# that usually indicates an uncorrectable
# parity or ECC memory error. 0=no | 1=yes
kernel.panic_on_unrecovered_nmi=1

# Panic if the kernel detects a soft-lockup
# error (1). Otherwise it lets the watchdog
# process skip it's update (0)
# kernel.softlockup_panic=0

# Panic on oops too. Use with caution.
# kernel.panic_on_oops=30
```

Load it:

```
sudo sysctl -p /etc/sysctl.d/panic.conf
```

[Source](https://www.supertechcrew.com/kernel-panics-and-lockups/)


## FTP

Install FTP Server:

```
sudo apt install -y vsftpd
```

Change the Configurations:

```
sudo vi /etc/vsftpd.conf
```

Uncomment the following line:
```
write_enable=YES
```

Restart `vsftpd`:

```
sudo systemctl restart vsftpd.service
```

## Create FTP user with Specific Directory Access

**FTP User:** `ftpuser`  
**Home Directory:** `/data/datalogger-data/`

```
sudo mkdir /data
sudo chmod 755 /data/
sudo mkdir -p /data/datalogger-data/

chown -R ftpuser:ftpuser /data/datalogger-data/
```

[Source](https://linuxroutes.com/create-ftp-user-with-specific-directory-access/)



## todo - need to update timezone; wasn't asked for in install

