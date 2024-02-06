This article is about how to develop a “compound trigger” action in openwhisk that only fires the next trigger when all the previous triggers arrives  (e.g. from git webhooks, or alarms).

["Compound-trigger" action](https://docs.google.com/drawings/d/1vuVv8oTUOf1VD017zIsQ6Jdoys8al-Zy_55RJZvDK2Y/edit)

The "compound trigger" action also interact with our storage server(e.g. mc ls flare/fcre/compound-trigger). Once it receives a trigger, it will update the "state.json" file in the storage server. For example, if the action is fired by a alarm trigger, it will update the "alarm" field from false to true. 

## Step 1: Build the compound-trigger image using the source code below and create the corresponding action. 

Source code: https://github.com/Jyuqi/FLARE_DEBUG_NODEJS/tree/master/functions/compound-trigger.

```sh
$ cd [parent dir of compound-trigger]
$ docker build -t flareforecast/openwhisk-compound-trigger -f compound-trigger/Dockerfile .
$ docker push flareforecast/openwhisk-compound-trigger
$ wsk action create compound-trigger --docker flareforecast/openwhisk-compound-trigger
```

## Step 2: Combine flare-noaa-ready-[lake] and flare-observations-ready-[lake] triggers and compound-trigger action as two rules. 
When either of  flare-noaa-ready-[lake] and flare-observations-ready-[lake] triggers is fired, the compound-trigger action would be invoked. The action updates state.json in storage server(e.g. mc cat flare/fcre/compound-trigger/state.json). If the action is invoked by flare-noaa-ready-[lake], it updates "noaa" field to "true"; If the action is invoked by flare-observations-ready-[lake], it updates "observations" field to "true". When two fields all turn to "true", it fires another trigger flare-drivers-ready-[lake], save the old state.json to state_{timestamp}.json and reinitiate a state.json with all fileds "false." Otherwise it will return developer error and block the next trigger from being fired. 

```sh
wsk trigger create flare-noaa-ready-fcre
wsk trigger create flare-observations-ready-fcre
wsk rule create compound-trigger-rule1 flare-noaa-ready-fcre compound-trigger
wsk rule create compound-trigger-rule2 flare-observations-ready-fcre compound-trigger
```

## payload configuration

The compound-trigger also uses flare-config.yml to get infomation for next trigger name and payload.
```yaml
## Openwhisk Settings
openwhisk:
  days-look-back: 1
  container-dependencies: ""
  next-trigger:
    name: flare-drivers-ready-fcre
    container_name: flare-generate-forecast
```

The payload for compound-trigger is stored in flare-config.yml of flare-process-observations and flare-process-noaa. Remember to include type field with value "noaa" or "observations". For example,
```yaml
## Openwhisk Settings
openwhisk:
  days-look-back: 0
  container-dependencies: "flare-download-observations"
  next-trigger:
    name: flare-observations-ready-fcre
    container_name: compound-trigger
    type: observations
```

To test compound-trigger as an action only,
```sh
$ wsk action invoke compound-trigger -P payload.json
```
The payload.json is exactly same with payload field above. ssh_key is optional

```json
{
    "type": "noaa/observations",
    "container_name": "compound-trigger",
    "lake": "fcre",
    "s3_endpoint": "xxx",
    "s3_access_key": "xxx",
    "s3_secret_key": "xxx",
    "openwhisk_apihost": "xxx",
    "openwhisk_auth": "xxx",
}
```
