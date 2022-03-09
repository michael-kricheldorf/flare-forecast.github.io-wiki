Ibmcloud deployment
===

## Ibmcloud cli installation

> **Download ibmcloud cli**

If your system is linux, use below command.

```shell
sudo curl -fsSL https://clis.cloud.ibm.com/install/linux | sh
```

If not, check with [offical documentation](https://https://cloud.ibm.com/docs/cli?topic=cli-getting-started) for installation.

> **Set your account**

```shell
ibmcloud login -u $YOUR_USERNAME -p $YOUR_PASSWORD -r $YOUR_ACCOUNT_REGION
```

> **Set your target**

Default

```shell
ibmcloud target --cf
```

Or target your personal workspace

```shell
ibmcloud target -o <value> -s <value>
```
* If you choose to target your personal workspace, you also need to change line 40 in github action yaml file.

## Install openwhisk package on ibmcloud

> **Install openwhisk package**

```shell
ibmcloud plugin install Cloud-Functions
```

> **Install openwhisk alarm package**

```shell
ibmcloud wsk package get /whisk.system/alarms --summary
```

## Set up flare sequence on ibmcloud

> **Set up action with flare functions**

```shell
ibmcloud wsk action create function1 --docker flareforecast/flare -t 600000 -m 2048
ibmcloud wsk action create function2 --docker flareforecast/flare -t 600000 -m 2048
ibmcloud wsk action create function3 --docker flareforecast/flare -t 600000 -m 2048
ibmcloud wsk action create function4 --docker flareforecast/flare -t 600000 -m 2048
```

> **Combine each action into sequence**

```
ibmcloud wsk action create flare --sequence function1,function2,function3,function4
```

* Make sure the sequence name is "flare"!


## Use your own json file ##
> **Store your secret keys in Github account**

Step 1: Click on the setting page in your repo

If setting page doesn't show, you need to contact the admin of the repo to give you admin access.

Step 2: Click on secrets in the left menu. Then, click on actions.

Step 3: Click on "New repository secret"(On the right top)

Step 4: Create secret keys (Make sure below variable are set)
* AWS_ACCESS_KEY_ID
* AWS_SECRET_ACCESS_KEY
* IBMCLOUD_USERNAME
* IBMCLOUD_PASSWORD
* IBMCLOUD_REGION

![](https://i.imgur.com/Gc2bu11.png)

## Upload your json file ##

Upload lake json file to this [repo](https://https://github.com/FLARE-forecast/deployed-forecasts)

The json file format needs to follow as below example
```json=
{
  "forecast_code": "XXX",
  "config_set": "XXX",
  "function": "1",
  "use_https": "XXX",
  "aws_default_region": "XXX",
  "aws_s3_endpoint": "XXX",
  "sim_name": "XXX"
}
```
* The "function" variable needs to be set to "1".

## Modify alarm's frequency
If you want to modify frequency of openwhisk alarm, you can modify the "cron" option at line 83 in github action yaml file.

``` shell=83
ibmcloud wsk trigger create $alarm_name --feed /whisk.system/alarms/alarm --param cron "0 */6 * * *"  --param trigger_payload ${payload}
```