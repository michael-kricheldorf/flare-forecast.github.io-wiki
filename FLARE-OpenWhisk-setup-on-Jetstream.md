# Overview

This document describes how to deploy an OpenWhisk setup on Jetstream from scratch. This setup is the simplest possible OpenWhisk "local" deployment - with one node (m1.medium, or larger) hosting the entire cluster, and using Jetstream S3 storage for data. The instructions follow the documentation for deployment of OpenWhisk using ansible in Ubuntu 18.04. Future documentation will consider a multi-node deployment.

The cluster is setup as follows:

* m1.medium instance
* 100GB volume
* a floating public IP address
* a security group that allows incoming ssh (port 22 for management), icmp (for ping), http and https (ports 80 and 443, for the OpenWhisk event API). 

Terminology:  

* host name: flare-frontend  
* Public IP address: PubIP  
* Private IP address: PrivateIP_frontend  

Private network/subnet on Jetstream:  

* Network: flarenet  
* Associated subnet: flaresubnet  
* Subnet: 10.1.1.0/24  
* Gateway: 10.1.1.1  

# Before getting started: obtain Jetstream API Access

[Contact XSEDE to request access to Jetstream API, following the guidelines](https://iujetstream.atlassian.net/wiki/spaces/JWT/pages/39682057/Using+the+Jetstream+API)

[Review documentation on API access](https://iujetstream.atlassian.net/wiki/spaces/JWT/pages/31391748/After+API+access+has+been+granted)

# Deploy flare-frontend instance (using Horizon GUI)

To avoid repeating documentation, [refer to this page and follow the steps there to create a network, security group, and instance using the Horizon GUI. Before you start the steps in the document below to deploy an instance for your cluster front-end, make note of the following:](https://iujetstream.atlassian.net/wiki/spaces/JWT/pages/44826638/Using+the+OpenStack+Horizon+GUI+Interface). 

* When launching an instance, select Ubuntu 18.04 from the list of JS-API-Featured
* Select the m1.medium flavor, and give it a 100GB volume
* Leave the default settings (create new volume: YES; delete volume on instance delete: NO)
* Name this instance “flare-frontend”
* As part of the process, you need to create a network. Call it “flarenet”. You may use the default 10.1.1.0/24 subnet (call it “flaresubnet”) with 10.1.1.1 as gateway address. 
* You will also need to request a floating public IP address, so the cluster has a stable public address for the OpenWhisk NGINX server. We’ll refer to this floating IP address as Pub_IP in the rest of the documentation; it will look something like 129.114.x.y (depending on what Jetstream public network you’re assigned)
* When creating security rules, ensure that you also add https, in addition to ssh and icmp as per the instructions
* After the flare-frontend instance is running, use the “Actions” drop-down to assign the floating IP address PubIP 

If all goes well, at the end of the process you should be able to ssh into your deployed instance with the key you provided:

ssh -i your_private_key ubuntu@Pub_IP

You should be able to see the status of the instance in the Horizon GUI under Project->Compute->Instances

Check and take note of the two IP addresses you see - you will need them later:

* PrivateIP_Frontend (a private address, looks like 10.1.1.xx) 
* PubIP (a public address, looks like e.g. 129.114.xx.yy) 

# Initial configuration of flare-frontend (using terminal+ssh)

ssh into the front-end from your computer, and install dependences including Docker:

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

# Setup CouchDB 

Here we will install CouchDB on the host. It can also be installed as a Docker container, but we’ll stick to a host install.

In a terminal in flare-frontend:

```
sudo apt-get install -y apt-transport-https gnupg ca-certificates
curl -L https://couchdb.apache.org/repo/bintray-pubkey.asc | sudo apt-key add -
echo "deb https://apache.bintray.com/couchdb-deb bionic main" | sudo tee -a /etc/apt/sources.list.d/couchdb.list
sudo apt-get update
sudo apt-get install couchdb -y
```

A text screen will pop up:

* Click Ok
* Then select Standalone, Ok
* Then enter your PrivateIP_frontend private address (it will look like 10.1.1.xx, as per above) as the interface to bind
* Click Ok
* Enter a strong password for your CouchDB admin. **Don't use special characters, though, as it can create problems with command-line arguments later on.** We’ll call this CouchDB_pwd in the rest of the document


## Sanity check that CouchDB is listening:

```
curl PrivateIP_frontend:5984/_node/_local/_config/ -u admin
```

## Set the reduce_limit as per OpenWhisk recommendation:

```
curl -X PUT http://PrivateIP_frontend:5984/_node/_local/_config/query_server_config/reduce_limit -d '"false"' -u admin
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
{"couchdb":"Welcome","version":"3.1.1","git_sha":"ce596c65d","uuid":"0faXXXXXXX91","features":["access-ready","partitioned","pluggable-storage-engines","reshard","scheduler"],"vendor":{"name":"The Apache Software Foundation"}}
```


# Install OpenWhisk on flare-frontend (using terminal+ssh):

Clone the OpenWhisk repository and perform initial setup, using the provided setup script:

```
cd
git clone https://github.com/openwhisk/openwhisk.git
cd openwhisk
cd tools/ubuntu-setup && ./all.sh
```

Configure OpenWhisk to use the CouchDB instance you just set up - make sure to replace PrivateIP_fronend and CouchDB_pwd appropriately:

```
cd ~/openwhisk/ansible
export OW_DB=CouchDB
export OW_DB_PROTOCOL=http
export OW_DB_HOST=PrivateIP_frontend
export OW_DB_PORT=5984
export OW_DB_USERNAME=admin
export OW_DB_PASSWORD=CouchDB_pwd
```

## Perform initial ansible setup:

The OpenWhisk git repo as of 03/23/2021 has a syntax error in a file used by ansible - you'll need to fix that manually. The file name is ~/openwhisk/ansible/group_vars/all - you need to look for a line with "default(1 second)", and remove the "second" string so it looks like: "default(1)"

```
cd ~/openwhisk/ansible
vi group_vars/all
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
db_host=PrivateIP_frontend
db_port=5984
```

## Change the maximum run time and memory of containers 

FLARE containers can run for longer than the default of 5 minutes. Let's set a generous timeout:

```
vi ~/openwhisk/common/scala/src/main/resources/application.conf
```

scroll down and change the time-limit max to 500 m (should be around line 435) and max memory to 4GB, and save:

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

## Deploy OpenWhisk containers

You will need to repeat these four steps after a reboot to deploy OpenWhisk

```
cd ~/openwhisk/ansible
ansible-playbook openwhisk.yml
ansible-playbook postdeploy.yml
ansible-playbook apigateway.yml
ansible-playbook routemgmt.yml
```


Now you should see all the containers running - docker ps should show containers for redis, kafka, apigateway, zookeeper, nginx, etc:

```
CONTAINER ID        IMAGE                                 COMMAND                  CREATED              STATUS              PORTS                                                                                                 NAMES
39821871f84a        openwhisk/action-nodejs-v10:nightly   "docker-entrypoint.s…"   About a minute ago   Up About a minute   8080/tcp                                                                                              wsk00_12_prewarm_nodejs10
fd243538e53b        openwhisk/apigateway:nightly          "/usr/bin/dumb-init …"   9 minutes ago        Up 9 minutes        80/tcp, 8423/tcp, 0.0.0.0:9000->9000/tcp, 0.0.0.0:9001->8080/tcp                                      apigateway
55c8dca89186        redis:4.0                             "docker-entrypoint.s…"   10 minutes ago       Up 10 minutes       0.0.0.0:6379->6379/tcp                                                                                redis
fc5979aec02c        nginx:1.19                            "/docker-entrypoint.…"   22 minutes ago       Up 22 minutes       0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp                                                              nginx
cf6c3791bc17        whisk/invoker:latest                  "/bin/sh -c 'exec /i…"   22 minutes ago       Up 22 minutes       0.0.0.0:17000->17000/tcp, 0.0.0.0:18000->18000/tcp, 0.0.0.0:12001->8080/tcp                           invoker0
16289d9747e2        whisk/controller:latest               "/bin/sh -c 'exec /i…"   28 minutes ago       Up 28 minutes       0.0.0.0:15000->15000/tcp, 0.0.0.0:16000->16000/tcp, 0.0.0.0:8000->2551/tcp, 0.0.0.0:10001->8080/tcp   controller0
ea7f490a7f77        wurstmeister/kafka:2.12-2.3.1         "start-kafka.sh"         29 minutes ago       Up 29 minutes       0.0.0.0:9072->9072/tcp, 0.0.0.0:9093->9093/tcp                                                        kafka0
4f1f60c4c74f        zookeeper:3.4                         "/docker-entrypoint.…"   29 minutes ago       Up 29 minutes       0.0.0.0:2181->2181/tcp, 0.0.0.0:2888->2888/tcp, 0.0.0.0:3888->3888/tcp                                zookeeper0
```

# Install wsk OpenWhisk command-line interface (CLI)

[Download the wsk CLI amd64 binary from the releases repository](https://github.com/apache/openwhisk-cli/releases), then copy it to /usr/local/bin and set properties to use the cluster:


```
cd 
wget https://github.com/apache/openwhisk-cli/releases/download/latest/OpenWhisk_CLI-latest-linux-amd64.tgz
tar -xzf OpenWhisk_CLI-latest-linux-amd64.tgz
chmod +x wsk
sudo cp wsk /usr/local/bin
wsk property set --apihost PrivateIP_frontend
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

Back to the flare-frontend node, let's install the alarm package, which creates timer events.

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
./gradlew :distDocker
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
$ docker run -d -p 11001:8080 -v $HOME/wsklogs/alarms:/logs \
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

# Configure SSL certificates

Now we will configure SSL for the public API endpoint, so that our OpenWhisk cluster can receive requests from sites such as github. 

## Roadmap

We'll use letsencrypt to get a free certificate. letsencrypt requires that a web server successfully responds to a challenge in order to issue a short-term certificate; this is a little tricky, as the OpenWhisk nginx server is not configured to do that. The instructions below do this by temporarily bringing down OpenWhisk, then run the certbot CLI tool to use the ACME protocol to grab a certificate from letsencrypt.

## Temporarily turn off nginx to request certificate

Let's kill our existing nginx and apigateway containers:

```
docker kill nginx
docker kill apigateway
```

## Install certbot from snap


```
cd
mkdir certbot
cd certbot
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

## Grab certificate from letsencrypt using certbot

Note: for this, you will need to find out the domain name that is mapped to your flare-frontend's PubIP. You can find this out by running: 

```
nslookup PubIP
```

It should look like this (replacing XXX, YYY, ZZZ, WWW with numbers which match the PubIP address) js-XXX-YYY-ZZZ-WWW.jetstream-cloud.org

```
cd ~/certbot
sudo certbot certonly --standalone
```


## Copy certificate/key and save old nginx configuration


```
cd ~/certbot
mkdir old-certs
cp ~/openwhisk/ansible/roles/nginx/files/openwhisk-server-cert.pem old-certs
cp ~/openwhisk/ansible/roles/nginx/files/openwhisk-server-key.pem old-certs
sudo bash
# cp /etc/letsencrypt/live/js-XXX-YYY-ZZZ-WWW.jetstream-cloud.org/fullchain.pem ~/openwhisk/ansible/roles/nginx/files/openwhisk-server-cert.pem
# cp /etc/letsencrypt/live/js-XXX-YYY-ZZZ-WWW.jetstream-cloud.org/privkey.pem ~/openwhisk/ansible/roles/nginx/files/openwhisk-server-key.pem
# chown ubuntu.ubuntu ~/openwhisk/ansible/roles/nginx/files/openwhisk-*.pem
```


## Restart OpenWhisk

```
cd ~/openwhisk/ansible
ansible-playbook openwhisk.yml
ansible-playbook postdeploy.yml
ansible-playbook apigateway.yml
ansible-playbook routemgmt.yml
```

## Check that it all works

From a browser, open: https://js-XXX-YYY-ZZZ-WWW.jetstream-cloud.org


# Configure GitHub to issue Webhook/API calls

[Instructions adapted from this 
document](https://github.com/apache/openwhisk-catalog/blob/master/packages/github/README.md)

## Create access token (using browser/GitHub)

[First, create a personal access token with the GitHub account](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/creating-a-personal-access-token)

** Make sure you copy and paste the token to a password manager - treat it like a password! You'll need to generate a new one if you lose it

While generating your personal access token, give it any available name (e.g. FLARE). Be sure to select the following scopes, which will allow you to use wsk to register an event on GitHub that will fire every time there is a commit to a repository:

* repo: repo:status to allow access to commit status
* admin:repo_hook: write:repo_hook to allow the feed action to create your webhook

## Create OpenWhisk trigger (using ssh/flare-frontend)

On flare-frontend, create a trigger for the GitHub push event type by using the myGit/webhook feed. 

First, create a package binding that is configured for your GitHub repository, and with the personal access token created in the previous step. Replace myGitUser with your GitHub user name, myGitRepo with your repository name, and accessToken you had saved from the GitHub page:

```
wsk -i package bind /whisk.system/github myGit \
  --param username myGitUser \
  --param repository myGitRepo \
  --param accessToken aaaaa1111a1a1a1a1a111111aaaaaa1111aa1a1a
```

Create a trigger for the GitHub push event type by using your myGit/webhook feed:

```
wsk -i trigger create myGitTrigger --feed myGit/webhook --param events push
```

## Update webhook (on GitHub)

The previous step has created a webhook on GitHub, but you need to edit it as follows before it works:

* Using your browser, go back to GitHub's page for the repository
* Click on the Settings tab
* Select Webhooks on the left pane
* Click the Edit button
* In the Payload URL box, you should see a URL with the format: undefined@10.1.1.xxx
* Replace this segment of the URL with a string of the format authkey:secret@DNSName - you can find authkey:secret from your ~/.wskprops file in flare-frontend, and DNSName with the name associate with PubIP (same you used for letsencrypt)
* The Payload URL should look like: https://d4******-****-****-****-**********80:fr***********************************************************qhQ@js-XXX-YYY-ZZZ-WWW.jetstream-cloud.org/api/v1/namespaces/_/triggers/myGitTrigger

## Test webhook

On flare-frontend, use these commands to create an action and poll activations:

```
wsk -i rule create myRule myGitTrigger /whisk.system/samples/greeting
wsk -i activation poll
```

In GitHub, commit a new file to the repository, and check that the event appears on the activation poll

## Deleting 

If you want to delete what you created for testing above:

```
wsk -i package delete myGit
wsk -i trigger delete myGitTrigger
```

# Add invoker nodes

TBD

# Miscellaneous notes, to-dos

## When ./gradlew distDocker failed

When it failed with the following message, the issue was a missing file in 

```
Step 19/20 : COPY openwhisk-server-key.pem /cert-gen
COPY failed: stat /var/lib/docker/tmp/docker-builder297477457/openwhisk-server-key.pem: no such file or directory
```

This is due to how we're setting up the letsencrypt cert, and file names.

In principle, I'd expect OW to instead look for privkey.pem for this file from:

```
/home/ubuntu/openwhisk/ansible/group_vars/all

    cert: "{{ nginx_ssl_server_cert | default('fullchain.pem') }}"
    key: "{{ nginx_ssl_server_key | default('privkey.pem') }}"
#    cert: "{{ nginx_ssl_server_cert | default('openwhisk-server-cert.pem') }}"
#    key: "{{ nginx_ssl_server_key | default('openwhisk-server-key.pem') }}"
```

But it didn't seem to? 

Perhaps better to just:

```
cd /openwhisk/ansible/roles/nginx/files
cp fullchain.pem openwhisk-server-cert.pem
cp privkey.pem openwhisk-server-key.pem
```

## Using step tool for certs

Using the [step](https://smallstep.com/docs/step-cli/basic-crypto-operations#get-a-tls-certificate-from-lets-encrypt) CLI tool seems like a much cleaner way to get a certificate from letsencrypt. 

Should try bringing nginx down, run step, copy certs to the right place, restart





