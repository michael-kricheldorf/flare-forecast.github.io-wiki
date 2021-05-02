# Open Deciosn Trees

1- Clone the "CIBR" repository, "decision-trees" branch containing the latest decision trees:

```bash
git clone --single-branch -b decision-trees https://github.com/FLARE-forecast/CIBR.git decision-trees
```

2- Go to [http://silverdecisions.pl/](http://silverdecisions.pl/).

3- Click on "Run".

4- Click on "Open existing diagram" button from the top left corner of the page.

5- Open `decision-trees/<decision-tree>.json` file.

# Save Decision Trees

1- Modify the decision tree as you wish.

2- Click on "Save current diagram" button from the top left corner of the page.

3- Save the file at `decision-trees/<decision-tree>.json`.

4- Move to the local Git repository from terminal:

```bash
cd decision-trees
```

5- Stage the changes in Git:

```bash
git add .

```

6- Commit the changes in Git:

```bash
git commit -m "Update Decision Tree"
```

7- Push the changes to the GitHub:

```bash
git push
```

# Scenario 1: Gateway Didn't Push and Not Accessible by CIBR Admin

## Resources
- [Cyberinfrastructure Maintenance Logs](Cyberinfrastructure-Maintenance-Logs)
- [Gateway Guidelines for Field Crew](Gateway-Guidelines-for-Field-Crew)
- [Gateway Troubleshooting](Gateway-Troubleshooting)

## Before Troubleshooting:
**Log:**
- Power LED Status
- LED1 Status
- Modem LEDs Status

## After Troubleshooting:
**Log:**
- Decision Tree Exit Code

## Default Decision
Contact CIBR admin; If not accessible, take gateway to the lab.

## Linux Commands for Troubleshooting

### Modem Recognized?

Check if the USB modem has been recognized by the operating system on the gateway.

**command to Run:**

```bash
lsusb
```

This command lists all the USB devices recognized by the operating system.

**What to Look for:**

```bash
Bus 002 Device 003: ID 1410:b020 Novatel Wireless
```

Our USB modems are recognized under the name of "Novatel Wireless". "Bus" number, "Device" number, and "ID" can be different from the sample output.

**Sample output:**

```bash
scc@mia:~$ lsusb
Bus 002 Device 003: ID 1410:b020 Novatel Wireless
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 003: ID 8087:0a2b Intel Corp. 
Bus 001 Device 002: ID 04b4:6570 Cypress Semiconductor Corp. 
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

### Modem Got IP?

Check if the USB modem was able to get an IP address after getting recognized by the operating system on the gateway.

**Command to Run:**

```bash
ip a
```

This command list all the network interfaces on the operating system.

**What to Look for:**

```bash
4: enx0015ff030033: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 1000
...
    inet 10.227.142.115/29 brd 10.227.142.119 scope global dynamic noprefixroute enx0015ff030033
```

"Novatel Wireless" modems' network interfaces start with `enx`. The actual name can be different from the sample output. Check if that interface have a line starting with `inet` following with an IP address such as `10.227.142.115/29`.

**Sample Output:**

```bash
scc@mia:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:01:c0:20:56:c6 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.5/24 brd 192.168.1.255 scope global enp2s0
       valid_lft forever preferred_lft forever
    inet6 fe80::201:c0ff:fe20:56c6/64 scope link 
       valid_lft forever preferred_lft forever
3: eno1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN group default qlen 1000
    link/ether 00:01:c0:20:56:cc brd ff:ff:ff:ff:ff:ff
4: enx0015ff030033: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 1000
    link/ether 00:15:ff:03:00:33 brd ff:ff:ff:ff:ff:ff
    inet 10.227.142.115/29 brd 10.227.142.119 scope global dynamic noprefixroute enx0015ff030033
       valid_lft 4437sec preferred_lft 4437sec
    inet6 2600:1003:b851:ee2b:5ceb:ff05:4812:dcc1/64 scope global deprecated dynamic mngtmpaddr noprefixroute 
       valid_lft 7167sec preferred_lft 0sec
    inet6 fe80::8347:b129:dd21:9407/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
5: wlp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 60:f6:77:df:23:07 brd ff:ff:ff:ff:ff:ff
    inet 10.42.0.1/24 brd 10.42.0.255 scope global noprefixroute wlp1s0
       valid_lft forever preferred_lft forever
    inet6 fe80::62f6:77ff:fedf:2307/64 scope link 
       valid_lft forever preferred_lft forever
6: ipop: <BROADCAST,MULTICAST,NOARP,UP,LOWER_UP> mtu 1280 qdisc fq_codel state UNKNOWN group default qlen 1000
    link/ether ae:83:95:7b:03:ea brd ff:ff:ff:ff:ff:ff
    inet 192.168.10.12/16 brd 192.168.255.255 scope global ipop
       valid_lft forever preferred_lft forever
    inet6 fd50:dbc:41f2:4a3c:b7e5:e403:6281:f1fd/64 scope global 
       valid_lft forever preferred_lft forever
    inet6 fe80::ac83:95ff:fe7b:3ea/64 scope link 
       valid_lft forever preferred_lft forever
```