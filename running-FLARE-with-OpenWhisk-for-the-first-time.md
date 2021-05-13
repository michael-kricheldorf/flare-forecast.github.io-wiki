**NOTE:** For running the commands, set `FLARE_CONTAINER_NAME` environment variable to the container name you are willing to run:
- flare-download-noaa (In Progress, Not Functional Yet)
- flare-process-noaa
- flare-download-observations
- flare-process-observations
- flare-generate-forecasts
- flare-visualize

For instance:

```bash
$ export FLARE_CONTAINER_NAME=flare-download-noaa
```

Make sure the environment variable has been set properly by printing that in the terminal

```bash
$ echo $FLARE_CONTAINER_NAME
```

The output should be the name of the variable you have already set:

```bash
flare-download-noaa
```

## Local wsk action Deployment

```sh
### if the openwhisk action is not created before, -t [time in ms] -m [memory in MB]
$ wsk action create $FLARE_CONTAINER_NAME --docker flareforecast/$FLARE_CONTAINER_NAME -t 18000000 -m 2048
### if the openwhisk action is already existed, only include updated parameters like below
$ wsk action update $FLARE_CONTAINER_NAME -t 18000000
```

**Invocation and Payload**

The payload.json should contain all the parameters we need to pass while invoking the action through the /run endpoint.
SSH key is optional . For example,

```json
{
    "FLARE_VERSION": "21.01.4/latest", 
    "container_name": "$FLARE_CONTAINER_NAME",
    "lake": "fcre",
    "s3_endpoint": "xxx",
    "s3_access_key": "xxx",
    "s3_secret_key": "xxx",
    "openwhisk_apihost": "xxx",
    "openwhisk_auth": "xxx"
}
```
Or with the sshkey, 
```json
{
    "FLARE_VERSION": "21.01.4/latest", 
    "container_name": "$FLARE_CONTAINER_NAME",
    "lake": "fcre",
    "s3_endpoint": "xxx",
    "s3_access_key": "xxx",
    "s3_secret_key": "xxx",
    "openwhisk_apihost": "xxx",
    "openwhisk_auth": "xxx",
    "ssh_key":["-----BEGIN RSA PRIVATE KEY-----","...","...","-----END RSA PRIVATE KEY-----"]}'
}
```

To invoke the action, you can either invoke by passing a json file or passing all parameters one by one.
```sh
$ wsk action invoke $FLARE_CONTAINER_NAME -P payload.json
```

**wsk Cheatsheet**
```sh
# invoke action with payload file
$ wsk action invoke $FLARE_CONTAINER_NAME -P payload.json
# After invocation, open another terminal to monitor whether the action finishes
$ wsk activation poll
# Access the activation log
$ wsk activation get [activation id]
$ wsk activation result [activation id]
# update an action with different memory limit or timeout limit setting, flare-process-observations requires 2G memory and flare-generate-forecast requires 512M memory
$ wsk action update $FLARE_CONTAINER_NAME -t [time in ms] -m [memory in MB]
# create an action based on dockerhub  image
$ wsk action create $FLARE_CONTAINER_NAME --docker flareforecast/$FLARE_CONTAINER_NAME -t [time in ms] -m [memory in MB]
```

## Compound trigger
compound-trigger has not been transferred flare repo yet. But it's already modified to match version 21.01.4. 
Code: https://github.com/Jyuqi/FLARE_DEBUG_NODEJS/tree/master/functions/compound-trigger
The payload for Compound trigger is a little different with the "type" field. Ideally, you don't need to manually invoke this action. The payload will be automatically generated when previous containers (flare-process-observations or flare-process-noaa) running flare_triggernext.sh. See the doc for more information: https://github.com/FLARE-forecast/flare-forecast.github.io/wiki/Build-a-%22compound-trigger%22-action-in-Openwhisk
```sh
$ wsk action create compound-trigger --docker flareforecast/openwhisk-compound-trigger
```

## wsk trigger deployment after creating all the actions
```sh
# create an every-3-hour alarm trigger
wsk trigger create flare-download-noaa-alarm-fcre --feed /whisk.system/alarms/alarm -p cron "0 */3 * * * " -p trigger_payload '{"FLARE_VERSION":"21.01.4", "container_name":"flare-download-noaa", "lake":"fcre", "s3_endpoint": "xxx", "s3_access_key":"xxx", "s3_secret_key":"xxx", "openwhisk_apihost": "xxx", "openwhisk_auth":"xxx"}'
wsk rule create flare-download-noaa-rule flare-download-noaa-alarm-fcre flare-download-noaa

wsk trigger create flare-download-noaa-ready-fcre
wsk rule create flare-process-noaa-rule flare-download-noaa-ready-fcre flare-process-noaa

# flare-data-repo-push-fcre should be a webhook
wsk rule create flare-download-observations-rule flare-data-repo-push-fcre flare-download-observations

wsk trigger create flare-download-observations-ready-fcre
wsk rule create flare-process-observations-rule flare-download-observations-ready-fcre flare-process-observations

wsk trigger create flare-noaa-ready-fcre
wsk trigger create flare-observations-ready-fcre
wsk rule create compound-trigger-rule1 flare-noaa-ready-fcre compound-trigger
wsk rule create compound-trigger-rule2 flare-observations-ready-fcre compound-trigger

wsk trigger create flare-drivers-ready-fcre
wsk rule create flare-generate-forecast-rule flare-drivers-ready-fcre flare-generate-forecast

wsk trigger create flare-forecast-ready-fcre
wsk rule create flare-visualize-rule flare-forecast-ready-fcre flare-visualize
```
