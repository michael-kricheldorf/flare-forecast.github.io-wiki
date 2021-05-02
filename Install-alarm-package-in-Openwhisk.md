We have set up Openwhisk on our sever, following steps here: https://github.com/FLARE-forecast/FLARE/wiki/Openwhisk-setup-on-local-environment. To use a timer like every 6 hours to trigger our actions , we also need to install alarm package of Openwhisk. The source code is here: https://github.com/apache/openwhisk-package-alarms. Here is the guide of how to deploy this alarm package.

## Step 1: Install alarm package with installCatalog.sh. 
```sh
export WSK_HOME="$HOME/openwhisk"
export OPENWHISK_HOME=$WSK_HOME
export AUTH_KEY=`cat ~/openwhisk/ansible/files/auth.whisk.system`
export DB_HOST=`awk -F "=" '/db_host/ {print $2}' $WSK_HOME/ansible/db_local.ini`
export DB_PORT=`awk -F "=" '/db_port/ {print $2}' $WSK_HOME/ansible/db_local.ini`
### look up your user name and password in $WSK_HOME/ansible/db_local.ini
export DB_URL=http://[your_username]:[your_password]@$DB_HOST:$DB_PORT
export DB_PREFIX=`awk -F "=" '/db.prefix/ {print $2}' $WSK_HOME/whisk.properties`
export EDGE_HOST=`awk -F "=" '/APIHOST/ {print $2}' $HOME/.wskprops`
export API_HOST=`awk -F "=" '/APIHOST/ {print $2}' $HOME/.wskprops`

# ./installCatalog.sh <authkey> <edgehost> <apihost> <workers> <dburl> <dbprefix>
./installCatalog.sh $AUTH_KEY $EDGE_HOST $API_HOST 2 $DB_URL $DB_PREFIX
```
EDGEHOST and APIHOST are the IP:port of your OpenWhisk controller.
To verify the installation, use following command.

```sh 
wsk -i package get --summary /whisk.system/alarms
wsk -i action get --summary /whisk.system/alarms/alarm
``` 

## Step 2:  Build the alarm provider docker image

```sh
./gradlew :distDocker
```

## Step 3: Run the alarm provider docker

```sh
set -e
set -x
WSK_HOME=$HOME/openwhisk
DB_PROTOCOL=`awk -F "=" '/db_protocol/ {print $2}' $WSK_HOME/ansible/db_local.ini`
DB_HOST=`awk -F "=" '/db_host/ {print $2}' $WSK_HOME/ansible/db_local.ini`
DB_PORT=`awk -F "=" '/db_port/ {print $2}' $WSK_HOME/ansible/db_local.ini`
DB_PREFIX=`awk -F "=" '/db.prefix/ {print $2}' $WSK_HOME/whisk.properties`

### look up your user name and password in $WSK_HOME/ansible/db_local.ini
$ docker run -d -p 11001:8080 -v $HOME/wsklogs/alarms:/logs \
-e PORT=8080 -e ROUTER_HOST=172.17.0.1 \
-e DB_PREFIX=${DB_PREFIX} -e DB_USERNAME=[your_username] \
-e DB_PASSWORD=[your_password] -e DB_HOST=${DB_HOST}:${DB_PORT} \
-e DB_PROTOCOL=${DB_PROTOCOL} whisk/catalog_alarms
```

The docker is running locally on port 11001, which is verified from docker ps
```sh
$ docker ps
CONTAINER ID        IMAGE                                 COMMAND                  CREATED              STATUS              PORTS                                                                                                 NAMES
2a1fc0060272        whisk/catalog_alarms                  "docker-entrypoint.s…"   4 weeks ago          Up 4 weeks          0.0.0.0:11001->8080/tcp                                                                               boring_noether

```

## Step 4: Ready to use it!
Here is an example of usage. If you're interested in more details, visit https://github.com/apache/openwhisk-package-alarms.
```sh
# open a terminal, poll the activation result
$ wsk -i activation poll 
# Open another terminal
# create a trigger, which would be fired every 8 seconds
$ wsk -i trigger create everyEightSeconds --feed /whisk.system/alarms/alarm -p cron "*/8 * * * * *" -p trigger_payload "{\"name\":\"Mork\", \"place\":\"Ork\"}"
# combine this trigger with an action using rule
$ wsk -i rule create myRule everyEightSeconds /whisk.system/samples/greeting
# now you can see the activation result in the first terminal
```
