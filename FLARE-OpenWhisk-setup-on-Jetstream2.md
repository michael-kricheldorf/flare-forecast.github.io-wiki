# Overview

This document describes how to deploy an OpenWhisk setup on Jetstream2 from scratch. This setup is the simplest possible OpenWhisk "local" deployment - with one node (m3.quad, or larger) hosting the entire cluster. The instructions follow the documentation for deployment of OpenWhisk using ansible in Ubuntu 18.04. Future documentation will consider a multi-node deployment.

The cluster is setup as follows:

* m3.quad instance
* 80GB volume
* a public IP address

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

Once your instance is up and running, it will show "Ready" in the Jetstream2 Exosphere GUI. Click on it to show the public IP address (e.g. 149.165.xxx.yyy - we'll refer to it as PubIP). Use your ssh key to log in and set an initial firewall that allows SSH, HTTP, and HTTPS only:

```
ssh -i your_private_key ubuntu@Pub_IP
sudo ufw allow 22/tcp
sudo ufw allow http
sudo ufw allow https
sudo ufw allow from 172.17.0.0/24
sudo ufw enable
```

# Initial configuration of flare_openwhisk (using terminal+ssh)

```
ssh -i your_private_key ubuntu@Pub_IP
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get install curl git python-pip python-setuptools build-essential libssl-dev libffi-dev python-dev software-properties-common -y
```


# Setup CouchDB 

Here we will install CouchDB on the host. In a terminal in flare_openwhisk:

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
* Then enter 172.17.0.1 as the interface to bind (this is the Docker IP address)
* Click Ok
* Enter a strong password for your CouchDB admin. **Don't use special characters, though, as it can create problems with command-line arguments later on.** We’ll call this CouchDB_pwd in the rest of the document


## Sanity check that CouchDB is listening:

```
curl 172.17.0.1:5984/_node/_local/_config/ -u admin
```

## Set the reduce_limit as per OpenWhisk recommendation:

```
curl -X PUT http://172.17.0.1:5984/_node/_local/_config/query_server_config/reduce_limit -d '"false"' -u admin
```

## Restart CouchDB:

```
sudo service couchdb restart
```

## Sanity check 

Check that you can access the database by issuing, from flare-frontend:

```
curl 172.17.0.1:5984
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

Configure OpenWhisk to use the CouchDB instance you just set up - **make sure to replace CouchDB_pwd appropriately with your CouchDB password**:

```
cd ~/openwhisk/ansible
export OW_DB=CouchDB
export OW_DB_PROTOCOL=http
export OW_DB_HOST=172.17.0.1
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
db_host=172.17.0.1
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

** These two ansible playbooks (initdb, wipe) should only be activated the first time you install the cluster, or you will lose data!!! **

```
cd ~/openwhisk/ansible
ansible-playbook initdb.yml
ansible-playbook wipe.yml
```

## Deploy OpenWhisk containers

You will need to repeat these four steps after a reboot to deploy OpenWhisk

```
cd ~/openwhisk/ansible
ansible-playbook openwhisk.yml
ansible-playbook postdeploy.yml
ansible-playbook apigateway.yml
ansible-playbook routemgmt.yml
```

Now you should see all the containers running - docker ps should show containers for redis, kafka, apigateway, zookeeper, nginx, etc.

# Install wsk OpenWhisk command-line interface (CLI)

[Download the wsk CLI amd64 binary from the releases repository](https://github.com/apache/openwhisk-cli/releases), then copy it to /usr/local/bin and set properties to use the cluster:


```
cd 
wget https://github.com/apache/openwhisk-cli/releases/download/latest/OpenWhisk_CLI-latest-linux-amd64.tgz
tar -xzf OpenWhisk_CLI-latest-linux-amd64.tgz
chmod +x wsk
sudo cp wsk /usr/local/bin
wsk property set --apihost 172.17.0.1
wsk property set --auth `cat ~/openwhisk/ansible/files/auth.guest`
```

And you can test the wsk CLI and run a simple task:

```
wsk -i action invoke /whisk.system/utils/echo -p message hello --result
```

If all goes well, you should see:

```
{
    "message": "hello"
}
```

# Use wskadmin to delete guest user and create FLARE user

The installation process creates a user account in the "guest" namespace:

```
cd ~/openwhisk/tools/admin
./wskadmin user list guest -a
```

Let us create a new user, FLARE, in a new namespace, FLARE:

```
./wskadmin user create FLARE -ns FLARE
./wskadmin user list FLARE -a
```

Now let's set the wsk CLI authentication key for this user; replace the argument after --auth with the key created for the FLARE user above:

```
wsk property set --auth XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX:YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY
```

Now you should be able to issue wsk commands as FLARE:

```
wsk property get --auth
wsk -i action invoke /whisk.system/utils/echo -p message hello --result
```

And you can delete the default guest account:

```
./wskadmin user delete guest -ns guest
```

# Install the OpenWhisk alarm package

Let's install the alarm package, which creates timer events.

```
cd
git clone https://github.com/apache/openwhisk-package-alarms.git
export WSK_HOME="$HOME/openwhisk"
export OPENWHISK_HOME=$WSK_HOME
export AUTH_KEY=`cat ~/openwhisk/ansible/files/auth.whisk.system`
export DB_HOST=`awk -F "=" '/db_host/ {print $2}' $WSK_HOME/ansible/db_local.ini`
export DB_PORT=`awk -F "=" '/db_port/ {print $2}' $WSK_HOME/ansible/db_local.ini`
### replace CouchDB_Password below
export DB_URL=http://admin:CouchDB_Password@$DB_HOST:$DB_PORT
export DB_PREFIX=`awk -F "=" '/db.prefix/ {print $2}' $WSK_HOME/whisk.properties`
export EDGE_HOST=`awk -F "=" '/APIHOST/ {print $2}' $HOME/.wskprops`
export API_HOST=`awk -F "=" '/APIHOST/ {print $2}' $HOME/.wskprops`
cd openwhisk-package-alarms
./installCatalog.sh $AUTH_KEY $EDGE_HOST $API_HOST 2 $DB_URL $DB_PREFIX
```

To verify the installation, use following commands:

```
wsk -i package get --summary /whisk.system/alarms
wsk -i action get --summary /whisk.system/alarms/alarm
```

Now build the Docker image for the alarm:

```
./gradlew distDocker
```

Now run the Docker image - again replace CouchDB_Password appropriately:

```
set -e
set -x
WSK_HOME=$HOME/openwhisk
DB_PROTOCOL=`awk -F "=" '/db_protocol/ {print $2}' $WSK_HOME/ansible/db_local.ini`
DB_HOST=`awk -F "=" '/db_host/ {print $2}' $WSK_HOME/ansible/db_local.ini`
DB_PORT=`awk -F "=" '/db_port/ {print $2}' $WSK_HOME/ansible/db_local.ini`
DB_PREFIX=`awk -F "=" '/db.prefix/ {print $2}' $WSK_HOME/whisk.properties`

### Substitute password here
docker run -d -p 11001:8080 -v $HOME/wsklogs/alarms:/logs \
-e PORT=8080 -e ROUTER_HOST=172.17.0.1 \
-e DB_PREFIX=${DB_PREFIX} -e DB_USERNAME=admin \
-e DB_PASSWORD=CouchDB_Password -e DB_HOST=${DB_HOST}:${DB_PORT} \
-e DB_PROTOCOL=${DB_PROTOCOL} whisk/catalog_alarms
```

The Docker container should now be running locally on port 11001, which is verified from docker ps:

```
$ docker ps | grep alarms
CONTAINER ID        IMAGE                                 COMMAND                  CREATED              STATUS              PORTS                                                                                                 NAMES
2a1fc0060272        whisk/catalog_alarms                  "docker-entrypoint.s…"   4 weeks ago          Up 4 weeks          0.0.0.0:11001->8080/tcp                                                                               boring_noether
```
