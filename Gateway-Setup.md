# Hardware
The gateway hardware is the [Fitlet2](https://fit-iot.com/web/products/fitlet2/).

### Carina:
**SN:** 1180222-01905  
**PN:** FITLET2-CJ3455-D2-M32S-FPCI-WI8260-J0-HTI-TE  
**TeamViewer ID:**  

### Mia
**SN:** 1180423-00507  
**PN:** FITLET2-CJ3455-D2-M32S-FPCI-WI8260-J0-HTI-TE  
**TeamViewer ID:**  

### Zoe
**SN:** 1180712-04018  
**PN:** FITLET2-CJ3455-D4-M64S-FPCI-WI8260-J0-HTI-TI  
**TeamViewer ID:** 331 560 795  

### Diana
**SN:** 1190117-11930  
**PN:** FITLET2-CJ3455-D4-M64S-FPCI-WI8260-J0-HTI-TI  
**TeamViewer ID:**  

### Bita
**SN:** 1190124-02963  
**PN:** FITLET2-CJ3455-D4-M64S-FPCI-WI8260-J0-HTI-TI  
**TeamViewer ID:** 357 419 025  

### Bjorn
**SN:** 1200227-00314  
**PN:** FITLET2-CJ3455-D4-M64S-FPCI-WI8260-J0-HTI-TE  
**TeamViewer ID:** 346 008 060 - 444 689 395  

### Annie 
**SN:** 1200227-00315  
**PN:** FITLET2-CJ3455-D4-M64S-FPCI-WI8260-J0-HTI-TE  
**TeamViewer ID:** 838 197 449  

## Hardware Specifications

**Dimensions:** 112mm * 84mm * 25mm (4.4" * 3.3" * 1")

**CPU:** Celeron J3455 [CJ3455]

**RAM:** 2 GB [D2] / 4 GB [D4]

**Storage:** M.2 SATA 32 GB [M32S] / M.2 SATA 64 GB [M64S]

**FACET-Card:** FC-PCI M.2 E-Key + mini-PCIe [FPCI]

**WiFi:** 802.11ac + BT (Intel 8260AC) [WI8260]

**mini PCIe modem:** Cellular antenna [J0]

**Top cover:** Industrial top-cover [HTI]

**Temperature range:** Extended temperature range -20°C to 70°C [TE] / Industrial temperature range: -40°C to 85°C [TI]

**DC input:** Standard DC input range 7V - 20V

# Software

## Operating System

Ubuntu Desktop 18.04 (64-bit)

## Bios

  

## Initial setup: BIOS and Ubuntu Installation on SD Card

What you will need:  

1- A computer with the ISO image of Ubuntu 18.04 downloaded.  
2- A USB drive with 4GB or more space to "burn" this ISO image from your computer. Note: this device will be erased and overwritten with the ISO image.   
3- Software in your computer to "burn" the ISO image to the USB disk; this depends on your O/S. [Here's one example of using dd with MacOS](https://www.raspberrypi.org/documentation/installation/installing-images/mac.md).  
4- A blank micro-SD card with 64GB or more space.  

Detailed steps:  

1- [Download the Ubuntu Desktop 18.04 image to your computer.](https://ubuntu.com/#download)  
2- Write the ISO image to the USB drive on your computer.  
3- Once the ISO image write is complete, remove the USB drive from your computer, and connect it to one of the USB ports of the fitlet2.  
4- Make sure the fitlet2 is powered off. Insert the blank SD card into the gateway.  Note: if you are using a SanDisk device, the colored/labeled side of the micro SD card should be facing the bottom of the fitlet2; make sure you insert until it latches.  
5- Connect USB mouse and keyboard to the fitlet2, and a cable to connect its HDMI output to a monitor.  
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
12- You should see a GRUB window; use the down arrow to select Install Ubuntu.  
13- The Ubuntu user interface will show up. Select English, English (US) keyboard; Continue.  
14- Select "Minimal installation", Continue.  
15- Select "Erase disk and install Ubuntu", Continue.  
16- Select the "MMC/SD card #1" device from drop-down list to install Ubuntu on the SD card.  
17- Click "Install Now", then Continue.  
18- Select timezone, user name and password, Continue. **Note** make sure you store username and password in a password manager in your computer, so you can retrieve it later.  
19- After the installation is complete, power off the fitlet2.   
20- Remove the USB drive, but keep the SD card in the fitlet2.  
21- Power up the fitlet2 - it should now boot from the SD card.  

### Bios Update

**Optional - when in doubt, do not try this!** if you need to update the BIOS, information is available [here](http://www.fit-pc.com/wiki/index.php?title=Fitlet2:_BIOS_Update).

## Upgrade Ubuntu

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
chmod 755 /data/
mkdir -p /data/datalogger-data/
chown -R ftpuser:ftpuser /data/datalogger-data/
```

[Source](https://linuxroutes.com/create-ftp-user-with-specific-directory-access/)

## Install OpenSSH Server

```
sudo apt install -y openssh-server
```

## Set up SSH

Generate SSH Keys:

```
ssh-keygen
```

Copy SSH keys for `root`, as well.

```
sudo cp -r ~/.ssh /root/
```

Add SSH public key to [your GitHub account](https://github.com/settings/keys).


For passwordless SSH login to the gateway, run the following command on the node from which you want to connect to the gateway:

```
ssh-copy-id remote-user@remote-node-ip
```

## Git

Install Git:

```
sudo apt install -y git
```

Increase the HTTP buffer and timeout limits to 500 MB and 10 minutes respectively:

```
git config --global http.postBuffer 524288000
git config --global http.timeout 600
```

Install Chromium depot_tools (It requires python 2.7 or 3.8 for python 3 support.):

```
mkdir -p ~/applications
cd ~/applications
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
cd depot_tools
git checkout chrome/3987
```

Install Python2 (For some reason, depot_tools doesn't work properly with Python3.):

```
sudo apt install -y python-minimal
```

add the following lines to `~/.bashrc`:

```
vi ~/.bashrc
```
```
# enable dept_tools
export PATH=~/applications/depot_tools:$PATH

# make depot_tools work with python2
export GCLIENT_PY3=0
```

Enable the changes to `~/.bashrc` without reboot:

```
source ~/.bashrc
```

## Monitor Network Traffic

Install vnStat:

```
sudo apt install -y vnstat
```

Create a DB for the interface in the first run:

```
sudo vnstat -u -i [interface]
```

Get the Statistics:

```
vnstat -i [interface]
```

## Update the Time

```
sudo apt install -y ntpdate
sudo ntpdate -buv ntp.ubuntu.com
```

## Change Timezone to EST

```
sudo timedatectl set-timezone EST
```

## Watchdog Configuration

[Source](http://www.fit-pc.com/wiki/index.php/Linux_Mint:_Watchdog_configuration)

## Controlling Front LEDs

[Source](http://www.fit-pc.com/wiki/index.php?title=Application_note_-_fitlet2_controlling_front_LEDs)

## Partition the Hard Drive

Follow the isntructions:

[Source](https://help.ubuntu.com/community/InstallingANewHardDrive)

Add label to the data partition:

```
sudo e2label /dev/sda1 data
```

Then edit the `fstab`:

```
sudo vi /etc/fstab
```

Add this line to the end:

```
/dev/disk/by-label/data /data auto nosuid,nodev,nofail,x-gvfs-show 0 0
```

Now it should look like this:

```
UUID=ab4dbe7c-1343-4bcf-a136-8bb15cc9bb5e /               ext4    errors=remount-ro 0       1
# /boot/efi was on /dev/mmcblk0p1 during installation
UUID=6C11-5CA7  /boot/efi       vfat    umask=0077      0       1
/swapfile                                 none            swap    sw              0       0
/dev/disk/by-label/data /data auto nosuid,nodev,nofail,x-gvfs-show 0 0
```

Then run:

```bash
sudo mkdir -p /data
sudo chown -R $USER:$USER /data
```

## Setting up Static IP Address for Gateway-Datalogger Ethernet Interface

**Interface:** enp2s0  
**IP Address:** 192.168.5.5  
**Netmask:** 255.255.255.0

**Datalogger IP Address** 192.168.5.2  
**Datalogger Netmask:** 255.255.255.0

[Source](https://linuxhint.com/setup_static_ip_address_ubuntu/)

## S-nail Mail Service

```
sudo apt install -y s-nail
```

## Controlling Front LEDs

[Source](http://www.fit-pc.com/wiki/index.php?title=Application_note_-_fitlet2_controlling_front_LEDs)

## TeamViewer

### Install TeamViewer

```
wget https://download.teamviewer.com/download/linux/teamviewer_amd64.deb
sudo apt install -y ./teamviewer_amd64.deb
```

### Set a Permanent Password

Set the permanent password at:

Options > Security > Personal password (for unattended access)

Then edit the following file:

```
sudo vi /etc/gdm3/custom.conf
```

Uncomment this line:

```
WaylandEnable=false
```

and Reboot.


## DNS Settings

In case the DNS settings need to be changed after setting up the networks and modem:

[DNS on Ubuntu](https://datawookie.netlify.app/blog/2018/10/dns-on-ubuntu-18.04/)


## Read-Only Filesystem

```
sudo apt install -y overlayroot
```

Edit the following file:

```
sudo vi /etc/overlayroot.conf
```

Add the following line:

```
overlayroot="tmpfs:swap=1,recurse=0"
```

Update Grub:

```
sudo update-grub2
```

Reboot to make it effective.

To apply changes to the read-only filesystem:

```
sudo overlayroot-chroot
```


## Set up Watchdog

**Note: this is for future reference, but not do this on fitlet2 with Ubuntu 18.04; we might revisit with Ubuntu 20.04

[Source](http://www.fit-pc.com/wiki/index.php/Linux_Mint:_Watchdog_configuration)
