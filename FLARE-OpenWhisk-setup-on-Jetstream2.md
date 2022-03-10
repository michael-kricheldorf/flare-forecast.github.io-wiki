# Overview

This document describes how to deploy an OpenWhisk setup on Jetstream2 from scratch. This setup is the simplest possible OpenWhisk "local" deployment - with one node (m3.quad, or larger) hosting the entire cluster. The instructions follow the documentation for deployment of OpenWhisk using ansible in Ubuntu 18.04. Future documentation will consider a multi-node deployment.

The cluster is setup as follows:

* m3.quad instance
* 80GB volume
* a public IP address
* a security group that allows incoming ssh (port 22 for management), icmp (for ping), http and https (ports 80 and 443, for the OpenWhisk event API). 

# Deploy flare_openwhisk instance (using Exosphere GUI)

To avoid repeating documentation, [refer to this page for documentation on the Exosphere GUI. Before you start the steps in the document below to deploy an instance for your cluster front-end, make note of the following:](https://docs.jetstream-cloud.org/ui/exo/exo/)

* When launching an instance, select Ubuntu 18.04 image
* Provide a name (e.g. flare_openwhisk)
* Select the m3.quad flavor (or larger)
* Select Custom disk size, and enter 80GB (or larger) in the text box
* Keep the 1 instance selection unchanged
* Under "Advanced options", click "Show"
* Under "Public IP address", select "Assign a public IP address to this instance"
* Choose your SSH key, or upload a new one

After you click "Create", the instance will be built and deployed within a few minutes

# Connect to flare_openwhisk instance (using SSH)

Once your instance is up and running, it will show "Ready" in the Jetstream2 Exosphere GUI. Click on it to show the public IP address (e.g. 149.165.xxx.yyy - we'll refer to it as PubIP). Use your ssh key to log in:

```
ssh -i your_private_key ubuntu@Pub_IP
```

# Initial configuration of flare_openwhisk (using terminal+ssh)

```
ssh -i your_private_key ubuntu@Pub_IP
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get install curl git python-pip python-setuptools build-essential libssl-dev libffi-dev python-dev software-properties-common -y
sudo pip install ansible==2.5.2
sudo pip install jinja2==2.9.6
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository  "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt-get install docker-ce docker-ce-cli containerd.io -y
sudo groupadd -f docker
sudo usermod -a -G docker $USER
```

Logout from flare-frontend, and log back in, so the Docker group addition takes effect. Sanity check that Docker is running:

```
exit
ssh -i your_private_key ubuntu@Pub_IP
docker ps
```

The command should work (but should not see no containers running yet)


