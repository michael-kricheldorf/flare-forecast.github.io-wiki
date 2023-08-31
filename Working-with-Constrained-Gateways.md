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

* Bypass DHCP for hostname resolution by adding the IP addresses of the hostnames to `/etc/hosts`.

* Use the "shallow Git clone" of a specific branch instead of the "full Git clone" of all branches:

```
git clone --single-branch --branch <branch-name> --depth 1 <repository>
```

* Consider the "shallow Git clone" of a specific branch instead of using "Git pull".

* Git fetch just the latest commit:

```
git fetch --depth 1 origin <branch-name>
git checkout FETCH_HEAD
```

_Note:_ This puts you in the "detached HEAD" state, making it harder to commit local changes to the local repository and push the commits to the remote repository. So it works best if you don't need to commit the local changes (e.g., for the `miscellaneous` repository).

* Increase the Git HTTP buffer and timeout limits to 500 MB and 10 minutes, respectively:
```
git config --global http.postBuffer 524288000
git config --global http.timeout 600
```