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
5- An Ethernet cable that provides a connection to the Internet using DHCP  

Detailed steps:  

1- [Download the Ubuntu Server 22.04 image to your computer.](https://ubuntu.com/download/server)  
2- Write the ISO image to the USB drive on your computer.  
3- Once the ISO image write is complete, remove the USB drive from your computer, and connect it to one of the USB ports of the fitlet2.  
4- Make sure the fitlet2 is powered off. Insert the blank SD card into the gateway.  Note: if you are using a SanDisk device, the colored/labeled side of the micro SD card should be facing the bottom of the fitlet2; make sure you insert until it latches.  
5- Connect USB mouse and keyboard to the fitlet2, and a cable to connect its HDMI output to a monitor, and the Ethernet cable. The instructions here assume you connected the Ethernet cable to the fitlet on the interface closest to the edge of the device (which is going to be enp1s0 during installation)  
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
13- After some initialization, the Ubuntu installation user interface will show up.  
14- Select English  
15- If it prompts to update the installer, select the default (Continue without updating).  
16- Select English, English (US) keyboard; Done.  
17- Use the up arrow and space bar to select "Ubuntu Server (minimized)", Done.  
18- The enp1s0 Ethernet network interface should  be listed as an interface this server can use, and it should show a DHCP-configured address. Check that the information is correct. Click on Done.  
19- The next screen asks for a proxy address; skip that by pressing Done.  
20- Also skip by pressing Done for mirror address.  
21- Leave "Use an entire disk" selected, and leave the default local disk selected - this will be your SD card (the name may vary depending on which SD card you use). Make sure you select the SD card as local disk - do not install on the FLEXXON_M2! Click Done.  
22- In the next screen you should see two devices: ubuntu-vg (new) and the FLEXXON_M2. Verify this is correct, click Done.  
23- In the "Confirm destructive action" (which means you'll overwrite the SD-card, which is ok); use down arrow then Continue.  
24- Enter server name, user name and password; **Note** make sure you store username and password in a password manager in your computer, so you can retrieve it later; Continue.  
25- Click Continue to "skip for now" upgrade to Ubuntu Pro.  
26- Use the space bar to select "Install OpenSSH Server"; navigate down to Done.  
27- In "Featured Server Snaps", skip and navigate down to Done.  
28- Now wait... After the installation is complete, power off the fitlet2.   
29- Remove the USB drive, but keep the SD card in the fitlet2.  
30- Power up the fitlet2 - it should now boot from the SD card.  

### Bios Update

**Optional - when in doubt, do not try this!** if you need to update the BIOS, information is available [here](http://www.fit-pc.com/wiki/index.php?title=Fitlet2:_BIOS_Update).

## Upgrade Ubuntu

When asked for which services to restart, enter 11 for none of the above

```
sudo apt update -y
sudo apt upgrade -y
sudo apt autoremove -y
```

## Install Vim, ping, killall, cron

```
sudo apt install -y vim
sudo apt install -y iputils-ping
sudo apt install -y psmisc
sudo apt install -y cron
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

## Prepare mount point

Check which disk is the SDD using sudo fdisk -l. For example, /dev/sda1. Create /data and mount it.

```
#if you need to partition the disk
sudo fdisk /dev/sda
# enter n to create a new
# accept defaults
# enter w to write
sudo mkfs.ext4 /dev/sda1
sudo mkdir /data
sudo vi /etc/fstab
#add an entry:
/dev/sda1 /data ext4 defaults 0 0
sudo mount /data
```

## Create FTP user with Specific Directory Access

**FTP User:** `ftpuser`  
**Home Directory:** `/data/datalogger-data/`

```
sudo chmod 755 /data/
sudo mkdir -p /data/datalogger-data/
sudo adduser --home /data/datalogger-data ftpuser
sudo chown -R ftpuser:ftpuser /data/datalogger-data/
```

[Source](https://linuxroutes.com/create-ftp-user-with-specific-directory-access/)


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

## Update the Time

```
sudo apt install -y ntpdate
sudo ntpdate -buv ntp.ubuntu.com
```

## Change Timezone to EST

```
sudo timedatectl set-timezone EST
```

## Configure Ethernet interface for field maintenance

This example assumes we're configuring the Ethernet port that is right next to the power supply cable on the fitlet (with name eno1) and we'll give it private address 172.16.100.1/24. To connect your laptop to it via Ethernet, manually configure your laptop's Ethernet interface with address 172.16.100.2/24 (i.e. subnet mask 255.255.255.0)

```
sudo vi /etc/netplan/99_config.yaml
# copy this into the configuration file; if you use an interface other than eno1, set its name instead
network:
  version: 2
  renderer: networkd
  ethernets:
   eno1:
      addresses:
        - 172.16.100.1/24
      routes:
        - to: default
          via: 172.16.100.1
      nameservers:
          addresses: [8.8.8.8]
# save the file then
sudo netplan apply
# check the interface is up
ip a
```

## Install nebula

```
sudo mkdir /etc/nebula
cd /etc/nebula
# copy the executables: nebula and run_nebula.sh
# copy the appropriate host's configuration, CA and host certificate, and host key: config.yaml, host.key, host.crt, ca.crt
# add this to the crontab to start on boot:
sudo crontab -e
@reboot /etc/nebula/nebula -config /etc/nebula/config.yaml
```

## Install Docker and EdgeVPN

```
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
# Make sure you are about to install from the Docker repo instead of the default Ubuntu repo:
apt-cache policy docker-ce
sudo apt install docker-ce
sudo systemctl status docker
sudo groupadd -f docker
sudo usermod -a -G docker $USER
```

[Source](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04)

```
cd
mkdir .evio
cd .evio
# copy the node's config.json here
echo openvswitch | sudo tee -a /etc/modules > /dev/null
sudo modprobe openvswitch
docker run -d -v /home/$USER/.evio/config.json:/etc/opt/evio/config.json -v /var/log/evio/:/var/log/evio/ --restart always --privileged --name evio-node --network host edgevpnio/evio-node:latest
```

[Source](https://edgevpn.io/trial/)


## Optional - install LoRa Rnode software

Build and install the tncattach software that allows using LoRa as an Ethernet NIC:

```
sudo apt install -y build-essential
git clone https://github.com/markqvist/tncattach.git
cd tncattach
make
sudo make install
```

[Source](https://github.com/markqvist/tncattach)

Install the rnodeconf software that allows configuring the rnode LoRa adapter:

```
sudo apt install -y python3-pip
pip3 install rns
```

Pip installs packages in /home/ubuntu/.local/lib/python3.10/site-packages by default, but we need to run rnodeconf as root. This is one solution to this issue (copying to the root account's site-packeges), but there may be cleaner approaches:

```
sudo mkdir -p /root/.local/lib/python3.10/site-packages
cd /home/ubuntu/.local/lib/python3.10/site-packages
sudo cp -r * /root/.local/lib/python3.10/site-packages
sudo chown -R ubuntu.ubuntu /root/.local/lib/python3.10/site-packages
```

With your LoRa rnode device connected to a USB port in the gateway, you can check its configuration:

```
cd /home/ubuntu/.local/bin
sudo ./rnodeconf /dev/ttyUSB0 -i
```
[Source](https://github.com/markqvist/rnodeconfigutil)

## Optional - set up LoRa interface with IP address

In the example below, the LoRa IP address is set up to 10.99.0.2/24 - adjust accordingly
The second line sets up tc to throttle the link

```
sudo bash
cd /usr/local/bin
vi restart_lora.sh
```

Add this to the restart_lora.sh script:

```
#!/bin/sh

/usr/bin/killall tncattach
/usr/local/sbin/tncattach /dev/ttyUSB0 115200 -d -e -n -m 400 -i $1
/usr/sbin/tc qdisc add dev tnc0 root tbf rate 20kbit burst 32kbit latency 400ms
```

Then:


```
chmod 755 restart_lora.sh
./restart_lora.sh 10.99.0.2/24
```

Add to the crontab entries for restarting at reboot and every hour:

```
@reboot /usr/local/bin/restart_lora.sh 10.99.0.2/24
@hourly /usr/local/bin/restart_lora.sh 10.99.0.2/24
```

## todo

* new instructions for ssh keys
* evio instructions
* add LoRa to crontab
* rest of instructions from 18.04 as needed



