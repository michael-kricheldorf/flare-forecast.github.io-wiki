Here are some tips for working with constrained gateways, such as gateways with unstable internet connections or internet connections over LoRa:

* Disable all unnecessary network traffic on the constrained interfaces (i.e., USB modem and LoRa radio interfaces).

* Turn off automatic (i.e., unattended) operating system updates.

Edit the following file:
```
sudo vi /etc/apt/apt.conf.d/20auto-upgrades
```

The file should look as follows:
```
APT::Periodic::Update-Package-Lists "0";
APT::Periodic::Unattended-Upgrade "0";
```

* Bypass DHCP for hostname resolution by adding the IP addresses of the hostnames to /etc/hosts.

* Use the "shallow Git clone" of a specific branch: git clone --single-branch --branch branch-name --depth 1 instead of the "full Git clone" of all branches.

* Consider the "shallow Git clone" of a specific branch instead of using "Git pull".

* Increase the Git HTTP buffer and timeout limits to 500 MB and 10 minutes, respectively:
```
git config --global http.postBuffer 524288000
git config --global http.timeout 600
```