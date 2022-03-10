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
```


# Setup CouchDB 

Here we will install CouchDB on the host. It can also be installed as a Docker container, but we’ll stick to a host install.

First, find out what **private** IP address your Jetstream2 instance is running; the output of this command will show it. It should be of the form 10.0.xxx.yyy; we'll refer to this as PrivateIP:

```
ifconfig ens3
```

In a terminal in flare_openwhisk:

```
sudo apt update && sudo apt install -y curl apt-transport-https gnupg
curl https://couchdb.apache.org/repo/keys.asc | gpg --dearmor | sudo tee /usr/share/keyrings/couchdb-archive-keyring.gpg >/dev/null 2>&1
source /etc/os-release
echo "deb [signed-by=/usr/share/keyrings/couchdb-archive-keyring.gpg] https://apache.jfrog.io/artifactory/couchdb-deb/ ${VERSION_CODENAME} main" \ | sudo tee /etc/apt/sources.list.d/couchdb.list >/dev/null
sudo apt-get update
sudo apt-get install couchdb -y
```

A text screen will pop up:

* Click Ok
* Then select Standalone, Ok
* Then enter your PrivateIP private address (it will look like 10.0.xxx.yyy, as per above) as the interface to bind
* Click Ok
* Enter a strong password for your CouchDB admin. **Don't use special characters, though, as it can create problems with command-line arguments later on.** We’ll call this CouchDB_pwd in the rest of the document


## Sanity check that CouchDB is listening:

```
curl PrivateIP:5984/_node/_local/_config/ -u admin
```

## Set the reduce_limit as per OpenWhisk recommendation:

```
curl -X PUT http://PrivateIP:5984/_node/_local/_config/query_server_config/reduce_limit -d '"false"' -u admin
```

## Restart CouchDB:

```
sudo service couchdb restart
```

## Sanity check 

Check that you can access the database by issuing, from flare-frontend:

```
curl PrivateIP_frontend:5984
```

You should see an output like:

```
{"couchdb":"Welcome","version":"3.2.1","git_sha":"244d428af","uuid":"774f11fbfa002485ca39e67824343a3b","features":["access-ready","partitioned","pluggable-storage-engines","reshard","scheduler"],"vendor":{"name":"The Apache Software Foundation"}}
```

# Install OpenWhisk on flare-frontend (using terminal+ssh):

Clone the OpenWhisk repository and perform initial setup, using the provided setup script:

```
cd
git clone https://github.com/openwhisk/openwhisk.git
cd openwhisk
cd tools/ubuntu-setup && ./all.sh
```

Configure OpenWhisk to use the CouchDB instance you just set up - **make sure to replace PrivateIP and CouchDB_pwd appropriately with your private IP and CouchDB password**:

```
cd ~/openwhisk/ansible
export OW_DB=CouchDB
export OW_DB_PROTOCOL=http
export OW_DB_HOST=PrivateIP
export OW_DB_PORT=5984
export OW_DB_USERNAME=admin
export OW_DB_PASSWORD=CouchDB_pwd
```

## Perform initial ansible setup:

```
cd ~/openwhisk/ansible
ansible-playbook setup.yml
```

Check that the file db_local.ini is populated with the variables for CouchDB as configured above - in particular, double-check the password is correct:

```
cat db_local.ini
[db_creds]
db_provider=CouchDB
db_username=admin
db_password=CouchDB_pwd
db_protocol=http
db_host=PrivateIP
db_port=5984
```

## Change the maximum run time and memory of containers 

FLARE containers can run for longer than the default of 5 minutes. Let's set a generous timeout of 500 minutes and increase memory limit to 4GB:

```
vi ~/openwhisk/common/scala/src/main/resources/application.conf
```

scroll down and change the time-limit max to 500 m (should be around line 476) and max memory to 4GB, and save:

```
    time-limit {
        min = 100 ms
        max = 500 m
        std = 1 m
    }

    # action memory configuration
    memory {
        min = 128 m
        max = 4096 m
        std = 256 m
    }
```

# Finish OpenWhisk install and deploy

Some of these steps may take several minutes...

## Install nodejs and npm, as prerequisites for OpenWhisk:

```
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt-get install -y nodejs
```

## Run the pre-requisites ansible playbook

```
cd ~/openwhisk/ansible
ansible-playbook prereq.yml
```

## Run Gradle

```
cd ~/openwhisk
./gradlew distDocker
```

## Initialize and clear the database

** These two ansible playbooks (initdb, wipe) should only be activated the first time you install the cluster, or you will lose data!!!

```
cd ~/openwhisk/ansible
ansible-playbook initdb.yml
ansible-playbook wipe.yml
```
