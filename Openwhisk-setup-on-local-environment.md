# Configuring the OpenWhisk server

This guide contains instructions on manually setting up Openwhisk and CouchDB on a fresh Ubuntu 18.04 server.

The guide is based on [the OpenWhisk Ansible README](https://github.com/openwhisk/openwhisk/blob/master/ansible/README.md#setup), which at the time of writing is missing some key steps and gotchas - hence this new guide.

## 1. Dependencies

```sh
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install curl git python-pip python-setuptools build-essential libssl-dev libffi-dev python-dev software-properties-common
sudo pip install ansible==2.5.2
sudo pip install jinja2==2.9.6
````
Install Docker:
```sh
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository  "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```


## 2. CouchDB

We're using a self-hosted CouchDB here, but you can choose to skip this step and use [an ephemeral CouchDB or Cloudant](https://github.com/openwhisk/openwhisk/blob/master/ansible/README.md#setup) instead.

```sh
sudo apt-get install -y apt-transport-https gnupg ca-certificates
curl -L https://couchdb.apache.org/repo/bintray-pubkey.asc | sudo apt-key add -
echo "deb https://apache.bintray.com/couchdb-deb bionic main" | sudo tee -a /etc/apt/sources.list.d/couchdb.list
sudo apt-get update
sudo apt-get install couchdb -y
sudo service couchdb start
```
Now you enter couchdb setup.
Setup it as **single node**. Set CouchDB interface bind address: 172.17.0.1. 
And also set password as administrator.

After installation and initial startup, you need to modify config file by using one of these methods.
1. visit Fauxton at http://127.0.0.1:5984/_utils#setup
In configuration page, Set the `reduce_limit` property to `false` (as per [OpenWhisk instructions](https://github.com/openwhisk/openwhisk/blob/master/ansible/README.md#persistent-couchdb)). 
Also set `bind_address=172.17.0.1` (if you have not set it before).
2. If the step 1 is not available to you, try this one.

Setting up admin user account is required. You can modify /opt/couchdb/etc/local.d/10-admins.ini
```sh
# Package-introduced administrative user
[admins]
;admin = <your couchdb admin password>    #insert this line
admin = ******
```
use below commands to modify configuration file 
```sh
### First check configuration file, and see which config varables you need to modified
curl 172.17.0.1:5984/_node/_local/_config/ -u admin:<your CouchDB password>

### Modify reduce_limit under query_server_config
curl -X PUT http://172.17.0.1:5984/_node/_local/_config/query_server_config/reduce_limit -d '"false"' -u admin:<your CouchDB password>

### If you don't set bind_address before, use below commands to modify.
curl -X PUT http://172.17.0.1:5984/_node/_local/_config/chttpd/bind_address -d '"172.17.0.1"' -u admin:<your CouchDB password>
```
After modify configs, restart couchDB service via `sudo service couchdb restart` to access CouchDB from Docker containers: running `curl <your CouchDB IP>:5984` should now output CouchDB info. My couchdb IP is: `172.17.0.1`

## 3. OpenWhisk

```sh
cd <folder which you want to install openwhisk>
git clone https://github.com/openwhisk/openwhisk.git
cd openwhisk
cd tools/ubuntu-setup && ./all.sh
```
First configure our database parameters:
```sh
cd <openwhisk home address>/ansible
export OW_DB=CouchDB
export OW_DB_PROTOCOL=http
export OW_DB_HOST=<the IP of your CouchDB server (should be accessible from Docker containers)>
export OW_DB_PORT=5984
export OW_DB_USERNAME=<your couchdb user>
export OW_DB_PASSWORD=<your couchdb password>

sudo ansible-playbook setup.yml
```
Now `db_local.ini` should contain the correct DB settings. 
As Openwhisk has default 5mins timeout setting is not enough for our project, we should modify it before building.
```sh
vi ~/openwhisk/common/scala/src/main/resources/application.conf +432
### modify line 432, in the time limit field under action timelimit configuration
max = 500 m
```
Let's continue with building OpenWhisk (it'll take a while):
```sh
### install nodejs and npm, as prerequisites for openwhisk
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt-get install -y nodejs

ansible-playbook prereq.yml

cd ~/openwhisk
sudo ./gradlew distDocker
```
> if "sudo ./gradlew distDocker" is not working, try to manage Docker as a non-root user. ([Issue #3429](https://github.com/apache/openwhisk/issues/3429))
Below is the link to solution.
[Manage Docker as a non-root user](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user)

Install openwhisk CLI to use Command-line Interface wsk
```sh
cd <folder you want to install openwhisk-cli>
git clone https://github.com/apache/openwhisk-cli.git
cd openwhisk-cli
sudo ./gradlew compile -PbuildPlatforms=linux-386
```
Simplify access to the wsk executable by moving or copying the wsk binary file to a path listed in your PATH environment variable. This will enable you to invoke the OpenWhisk CLI by just typing 'wsk' on the command line from anywhere.
```sh
sudo cp ~/openwhisk-cli/build/linux-386/wsk /usr/local/bin/wsk
wsk property set --apihost 172.17.0.1
wsk property set --auth `cat ~/openwhisk/ansible/files/auth.guest`
```

Then continue deploying it, these steps may take a long time:
```sh
cd ~/openwhisk/ansible
sudo service couchdb stop
sudo ansible-playbook couchdb.yml
sudo ansible-playbook initdb.yml
sudo ansible-playbook wipe.yml
sudo ansible-playbook openwhisk.yml

# installs a catalog of public packages and actions
sudo ansible-playbook postdeploy.yml

# to use the API gateway
sudo ansible-playbook apigateway.yml
sudo ansible-playbook routemgmt.yml
```

Now you're all set! To test Openwhisk, run following commands.
```sh
wsk -i property get
### try to invoke something
wsk -i action invoke /whisk.system/utils/echo -p message hello --result
```

