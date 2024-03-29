# Software for edge gateways

**Notes:**

We're storing scripts/configuration for edge gateways in the [miscellaneous](https://github.com/FLARE-forecast/miscellaneous) repo

We're storing an encrypted [pwsafe](https://pwsafe.org/)-compatible file with various keys/configuration files in a private [Google drive folder](https://drive.google.com/drive/folders/1sSO4UIZR0fnqKW-99Tr79nsCv9WY4aa6)

## Operating System

Ubuntu Server 22.04 (64-bit)

## Bios

### Initial setup: BIOS and Ubuntu Installation on SD Card

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

## Access via SSH

Since Ubuntu Server doesn't have a graphical user interface, to make the rest of the process easier, instead of accessing the gateway directly and typing the rest of the commands in Terminal on the gateway, you should find out the IP address of the gateway and establish an SSH connection to that via another computer that has a graphical user interface. Then you can access the gateway through that machine and access this documentation via the web browser on the second machine and copy and paste the commands to the gateway.

### Get IP Address

Find the IP address of the gateway from the list of network interfaces:

```
ip a
```

### Access the Gateway via SSH

Connect to the gateway via SSH from the terminal on the second machine:

```
ssh ubuntu@<gateway_ip_address>
```

## Upgrade Ubuntu

When asked for which services to restart, enter 11 for none of the above

```
sudo apt update -y
sudo apt upgrade -y
sudo apt dist-upgrade -y
sudo apt autoremove -y
```

## Install required tools and packages

```
sudo apt install -y vim iputils-ping psmisc cron tcpdump curl vsftpd ftp git autossh bash-completion apparmor-utils
sudo wget https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64 -O /usr/bin/yq &&\
    sudo chmod +x /usr/bin/yq
```

**Note:** [yq snap notes when running with sudo](https://github.com/mikefarah/yq#snap-notes)

## Enable bash-completion

1- Edit `~/.bashrc`:

```
echo 'if [ -f /etc/bash_completion ]; then . /etc/bash_completion; fi' >> ~/.bashrc
```

2- Run the following command:
```
source /etc/bash_completion
```

## Configure sudo to run without a password prompt

1- Run the following command to edit the sudoers file using the visudo command, which provides syntax checking and prevents you from saving invalid changes:

```
sudo visudo
```

2- This will open the sudoers file in the default text editor. Look for the line that contains the following:

```
%sudo   ALL=(ALL:ALL) ALL
```

3- Below that line, add the following:

```
ubuntu   ALL=(ALL) NOPASSWD:ALL
```

This line allows the `ubuntu` user to run any command with sudo without entering a password.

4- Save the changes and exit the text editor.

5- Verify that the sudo configuration is correct by running a command with sudo:

```
sudo ls
```

It should execute the command without asking for a password.


## Set password for `root` user

```
sudo -i
passwd
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

We need to log out and then log back in to notice the new hostname.

**Note: **Without updating `/etc/hostname` and `/etc/hosts`, this change is not permanent and will be undone after reboot.


## Disable Auto Updates and Unattended Upgrades

Edit the following file:

```
sudo vi /etc/apt/apt.conf.d/20auto-upgrades
```

The file should look as follows:

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


## Prepare mount point

Check which disk is the SDD using `sudo fdisk -l`. For example, `/dev/sda1`. Create `/data` and mount it.

If you need to partition the disk:

```
sudo fdisk /dev/sda
```

Enter n to create a new
Accept defaults
Enter w to write

```
sudo mkfs.ext4 /dev/sda1
sudo mkdir /data
sudo vi /etc/fstab
```

Add an entry to `/etc/fstab`:

```
/dev/sda1 /data ext4 defaults 0 0
```

```
sudo mount /data
```

Then change the ownership of the directory to `ubuntu` so that it is accessible by this non-root user.

```
sudo chown -R ubuntu:ubuntu /data
```


## FTP

Add and configure `ftpuser`:

```
sudo adduser --home /data/datalogger-data ftpuser
sudo usermod -a -G ftpuser ubuntu
mkdir /data/datalogger-data
mkdir /data/datalogger-raw-data
sudo chmod -R 775 /data/datalogger-data
sudo chmod -R 775 /data/datalogger-raw-data
sudo chown -R ftpuser:ftpuser /data/datalogger-data
sudo chown -R ftpuser:ftpuser /data/datalogger-raw-data

```

Change the configurations:

```
sudo vi /etc/vsftpd.conf
```

Uncomment the following line:

```
write_enable=YES
```

Add the following lines:

```
# Adjust transferred file permission
chmod_enable=YES
local_umask=002
```

Restart `vsftpd` service:

```
sudo systemctl restart vsftpd.service
```

Check `vsftpd` service:

```
systemctl status vsftpd.service
```

Test FTP service:

```
FTP localhost
```

Check FTP logs:

```
tail /var/log/vsftpd.log
```

## Git

Increase the HTTP buffer and timeout limits to 500 MB and 10 minutes respectively:

```
git config --global http.postBuffer 524288000
git config --global http.timeout 600
```

Set your Git account's default identity for both `ubuntu` user with and without `sudo`:

```
git config --global user.email "v.daneshmand@gmail.com"
git config --global user.name "vahid-dan"
sudo git config --global user.email "v.daneshmand@gmail.com"
sudo git config --global user.name "vahid-dan"
``` 

Also, [add your SSH public key to your GitHub account](https://github.com/settings/keys)


## Change Timezone to EST

```
sudo timedatectl set-timezone EST
```
## `dmesg` without root

To access system logs via `dmesg` without need to run the commands with root access:

```
echo "kernel.dmesg_restrict = 0" | sudo tee -a /etc/sysctl.conf > /dev/null
sudo sysctl -p
```

## Skip network interface waiting for configuration at startup + Add USB modem network interface configuration

0- Permissions for the network configuration files are too open by default. We should change them:

```
sudo chmod 600 /etc/netplan/*.yaml
```

1- Edit the network configuration file:

```
sudo vi /etc/netplan/00-installer-config.yaml
```

2- Add `optional: true` to interfaces. Also, add the USB modem network interface configuration. The output should be similar to the following:

```
# This is the network config written by 'subiquity'
network:
  ethernets:
    eno1:
      dhcp4: true
      optional: true
    enp1s0:
      dhcp4: true
      optional: true
    enp2s0:
      dhcp4: true
      optional: true
    enx0015ff030033:
      dhcp4: true
      optional: true
    enx0015ff080733:
      dhcp4: true
      optional: true
    enx0015ff025968:
      dhcp4: true
      optional: true
    enx0015ff357651:
      dhcp4: true
      optional: true
  version: 2
```
**Note 1: **The name of one of the network interfaces may be either `enp1s0` or `enp2s0` of Fitlet2 devices.

**Note 2: **The exact USB modem network interface name may be different. It should start with `enx`. Run `ip a` to find it from the list of interfaces. Even if you don't have any USB modem connected to the gateway, you still should add the above interfaces to the config file in case you attach a USB modem in the future.

3- Apply the changes.

```
sudo netplan apply
```

## Configure Ethernet interface for data logger connection and field maintenance

This example assumes we're configuring the Ethernet port that is right next to the power supply cable on the Fitlet2 (with name eno1) and we'll give it private address 10.10.1.2/24. To connect your laptop to it via Ethernet, manually configure your laptop's Ethernet interface with address 10.10.1.3/24 (i.e. subnet mask 255.255.255.0). The data logger IP address is 10.10.1.1/24.

```
sudo vi /etc/netplan/99_config.yaml
```

Copy this into the configuration file; if you use an interface other than eno1, set its name instead

```
network:
  version: 2
  renderer: networkd
  ethernets:
    eno1:
      dhcp4: no
      addresses: [10.10.1.2/24]
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

Save the file, then run netplan apply and check that the interface is up:

```
sudo netplan apply
ip a
```

## Install SSH key

Add these public key to ~/.ssh/authorized_keys:

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC0F2x12xyL357P/JiTZzlsa6Kfe4PL07VyW/xf7iocG/1YoZd3QOIpFJ1x4YIMA0YU2VoSi9tecE2mzbudcIm06VbpeXc59fMp5UH0KY8DpSzpk58eLvB78UOsMQ99swiTqYexUHqEfPaEmJfgkwkPvgzuKIcmJkRZuRGb7LurfxTXv5kGSJTJmdhWwSfdAgCJhTzzvGBYFzoC2SkvCsc8O6EGXKRdnoReJ0kSTI8bv2n7cAkMW5LQZPCxEAPsHb2fEbeU3kXqYJV025rWb1Xllp2fyIfy9dzUJP9IOAwdMDFBB+on2HF629DHD879MbkOWmC2/N2tcJ38SRQc8TPNKuiRrfsiuFjNvkghrhSsDYsYLWkP3gpDJkCMjos3To+hamKml3RnfGayPECUSM551kE3ClXwYqUuq9iiaJaMI3UAzKqVRYcO0V4/S20H8hq9vWJjWqw8vJzqzMETrf412a/izvI31LMFNnBa+Il5Z7bxLDsmnMQxsihwuk7i19M= renato@Renatos-MacBook-Pro-2.local
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDTf1GNFsGaG5iW2JycihXPL+aqVe3HkxUWlnWiQVtXBR7dijTQbDhx0B7D/QreVK57LEyKJsxP3UQhbS+fIwjS2FNV+rXCRpA6SSm7B9O0x+18chEpJx5G9vGHZOP5W+6HAU2jxpSpfPu3V9tdAfc1wkyZZSUs0t6vnD+BLOPxp6xGz+4FbuV0zB6OQvxk+PusPYwkeWxBQko/uDOg5peLPOmwRGNzGCfJ7Bdxgo9gAeUsUzXh1ntKisI6PcHepgZ5Hg7PQS1FA8IYcyTQbvB5say5mgooyFjCXFtzj6WfFDsonhfG2XZFdVv2JY98241jHdC7CmaHOlI6C07yNBxj ubuntu@rightly-merry-cougar
```

## Install nebula

First, clone the miscellaneous repo and download the nebula binary, then install executables in their right places:

```
cd
git clone https://github.com/FLARE-forecast/miscellaneous
cd miscellaneous/nebula
wget https://github.com/slackhq/nebula/releases/download/v1.7.1/nebula-linux-amd64.tar.gz
tar -xzvf nebula-linux-amd64.tar.gz
chmod 755 nebula restart_nebula.sh
sudo mkdir /etc/nebula
sudo cp nebula /etc/nebula
sudo cp config.yaml /etc/nebula
sudo cp restart_nebula.sh /usr/local/bin
```

You need to obtain the appropriate ca.crt, host.key and host.crt from the password safe for this Nebula IP address. Copy and paste the ca.crt, host.key and host.crt from the password safe to /etc/nebula then:

```
cd /etc/nebula
sudo chmod 600 ca.crt host.key host.crt
```

# Remove tcpdump from AppArmor Profiles

Having `tcpdum` in AppArmor profiles prevents you from using `tcpdump -w` to write the output file in `/data`.

First, check if `tcpdump` is in AppArmor profiles:

```
sudo aa-status | grep tcpdump
```

If so, you may want to remove it:

```
sudo aa-disable tcpdump
```


## Install Docker and EdgeVPN

First, install the latest Docker:

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

The install EdgeVPN - you'll need a proper config.json file for your node that you can also obtain from the password safe storing secrets. 

This configures docker to always restart the container, so there's no need for a crontab entry:

```
cd
mkdir .evio
cd .evio
# copy the node's config.json here
echo openvswitch | sudo tee -a /etc/modules > /dev/null
sudo modprobe openvswitch
docker pull edgevpnio/evio-node:latest
docker run -d -v /home/$USER/.evio/config.json:/etc/opt/evio/config.json -v /var/log/evio/:/var/log/evio/ --restart always --privileged --name evio-node --network host edgevpnio/evio-node:latest
```

[Source](https://edgevpn.io/trial/)


## Install LoRa Rnode software

Build and install the tncattach software that allows the use of LoRa as an Ethernet NIC:

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

If this works, configure the rnode device with LoRa parameters we currently use in the project:
```
sudo /home/ubuntu/.local/bin/rnodeconf /dev/ttyUSB0 -T --freq 903000000 --bw 62500 --txp 17 --sf 7 --cr 5
```

[Source](https://github.com/markqvist/rnodeconfigutil)

### Check LoRa Radio USB Connection

To verify if the LoRa radio is connected properly through a USB port, run the following command:

```
lsusb
```

In the outputs you should see a line similar to the following:

```
Bus 001 Device 004: ID 0403:6015 Future Technology Devices International, Ltd Bridge(I2C/SPI/UART/FIFO)
```

## Skip DNS Lookup by DHCP

For LoRa connections, it is essential to lower the internet connection load, e.g. DNS lookup by the DHCP.

Edit the following file:

```
sudo vi /etc/hosts
```

Add the following lines to skip DNS lookup for some domains:

```
140.82.114.3 github.com www.github.com
178.63.26.145 hc-ping.com
185.125.190.56 ntp.ubuntu.com
```

**Note:** The IP addresses may have been changed since the last update to this document. Make sure the IP addresses work. 