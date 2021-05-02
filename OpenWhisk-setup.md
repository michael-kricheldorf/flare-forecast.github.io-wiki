# Documentation for OpenWhisk setup at ACIS

## OpenWhisk Setup on Ubuntu 18.04.4 LTS**

Reference: https://github.com/apache/openwhisk/tree/master/tools/ubuntu-setup

### Environment Setup and git clone

```
sudo apt-get install git -y

git clone https://github.com/apache/openwhisk.git openwhisk

sudo apt install openjdk-8-jre-headless

cd openwhisk

cd tools/ubuntu-setup && ./all.sh

sudo apt-get install python-software-properties

sudo apt-get install nodejs

sudo apt install npm
```

### Build

```
sudo ./gradlew distDocker
```

### OpenWhisk Deploy

Reference: https://github.com/apache/openwhisk/blob/master/ansible/README.md

### Setting public domain name: -

Set public ip in “/home/harshit/openwhisk/ansible/group_vars/all” file by setting variable whisk_api_host_name to the public address of the web server 

### Configuring SSL: -

* Get the SSl certificate from certbot

 https://certbot.eff.org/lets-encrypt/ubuntubionic-nginx

* Copy the fullchain.pem and privkey.pem obtained from certbot to openwhisk/ansible/roles/nginx/files

* Update the nginx certificate section on line number 235 & 236 in file openwhisk/ansible/group_vars/all to our certificate name(We can also do it by setting environment variable also)

cert: "{{ nginx_ssl_server_cert | default('fullchain.pem') }}"

key: "{{ nginx_ssl_server_key | default('privkey.pem') }}"

* If you are setting certificate after initial setup then just run these commands after updating the certificate in group_vars/all file ansible-playbook setup.yml, ansible-playbook openwhisk.yml, ansible-playbook postdeploy.yml, ansible-playbook apigateway.yml, ansible-playbook routemanagement.yml(otherwise skip this step) 

### Setting CouchDb database on host and linking it with our openwhisk: 

* Install Couchdb on host using Cluster setting. Follow the below url for help -

  https://linuxize.com/post/how-to-install-couchdb-on-ubuntu-18-04/

* Set the following environment variables before running ansible-playbook

export OW_DB=CouchDB

export OW_DB_USERNAME=<your couchdb user>

export OW_DB_PASSWORD=<your couchdb password>

export OW_DB_PROTOCOL=<your couchdb protocol>

export OW_DB_HOST=<your couchdb host>

export OW_DB_PORT=<your couchdb port>

* After running setup.yml you can verify that database setting got updated properly in openwhisk/ansible/db_local.ini file if not you can update this file before proceeding further

* Current setting is as follows: -

db_provider=CouchDB

db_username=**********

db_password=**********

db_protocol=http

db_host=XXX.XXX.XXX.XXX

db_port=5984

### Steps:

```
sudo apt-get install python-pip

sudo pip install ansible==2.5.2

sudo pip install jinja2==2.9.6

ansible-playbook setup.yml

ansible-playbook couchdb.yml

ansible-playbook initdb.yml

ansible-playbook wipe.yml

ansible-playbook openwhisk.yml

ansible-playbook postdeploy.yml

ansible-playbook apigateway.yml

ansible-playbook routemgmt.yml
```

### OpenWhisk Cli Configuration

Reference: https://github.com/apache/openwhisk/blob/master/docs/cli.md

The API host can be acquired from the edge.host property in whisk.properties file.

### Steps:

```
./bin/wsk property set --apihost 172.17.0.1

./bin/wsk property set --auth cat ansible/files/auth.guest
```

### Starting Openwhisk after a host reboot

The following steps are needed to start Openwhisk after a host reboot in cibr-flare1:

```
ansible-playbook openwhisk.yml

ansible-playbook postdeploy.yml

ansible-playbook apigateway.yml

ansible-playbook routemgmt.yml
```

And in cibr-flare3 (couchdb):

```
sudo service couchdb restart
```

### Comments:

* I faced one issue while running ansible-playbook openwhisk.yml command. I fixed this issue by using command - sudo iptables -P INPUT ACCEPT

* You need to run initdb.yml every time you do a fresh deploy CouchDB to initialize the subjects database.

* The wipe.yml playbook should be run on a fresh deployment only, otherwise actions and activations will be lost(Important).

* Run postdeploy.yml after deployment to install a catalog of useful packages.

* To use the API Gateway, you'll need to run apigateway.yml and routemgmt.yml.

* Use ansible-playbook -i environments/<environment> openwhisk.yml to avoid wiping the data store. This is useful to start OpenWhisk after restarting your Operating System(Important).

* Currently I am storing openwhisk temp files at /home/harshit/openwhisk_temp location. You can use it for checking all the openwhisk's component configuration and logs(for example Nginx logs and configuration is here)

* /home/harshit/letsencrypt this folder contains all certbot files generated during certificate creation.


# GitHub Integration for Openwhisk

* Generate a GitHub personal access token

* While creating your personal access token, be sure to select the following scopes:

* * repo: repo:status to allow access to commit status

* * admin:repo_hook: write:repo_hook to allow the feed action to create your webhook

* Create a trigger for the GitHub push event type by using your myGit/webhook feed. The parameters are as follows:

* * username: The user name of the GitHub repository

* * repository: The GitHub repository

* * accessToken: Your GitHub personal access token

* * events: The GitHub event type of interest

wsk trigger create myGitTrigger --feed /whisk.system/github/webhook --param username <your username> --param repository <your repository name> --param accessToken <your access Token> --param events <event name>

## Example:-

wsk trigger create myGitTrigger --feed /whisk.system/github/webhook --param username harshitagrawal91 --param repository openwhisk_git_test --param accessToken [token] --param events push

# Running Action In different Docker Container:

Please refer below links:-

**Docker Action Documentation:** https://github.com/apache/openwhisk/blob/master/docs/actions-docker.md  
