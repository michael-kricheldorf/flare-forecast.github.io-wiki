# Introduction and overview of tutorial

During the 2022 CIBR virtual all-hands meeting, participants will go over a tutorial on how to configure and request execution of a set of retroactive forecast runs using FLARE. This is a capability that can facilitate CIBR researchers to execute batches of retroactive forecast runs. For example, let's say a researcher comes up with a new enhancement to the GLM model or FLARE data assimilation and would like to compare how it fares against a current approach using FCRE as a case study. They will be able to use what they learned in this tutorial to run retroactive forecasts for FCRE using current and new approach, say, for the entirety of July 2022.

In this tutorial, you will learn:

* An overview of the workflow, in particular: where the code, data, and configuration reside, and how the execution takes place using containers
* How to navigate data used by the workflow stored in cloud storage - S3 buckets
* How to create your own "starter lake" by starting from an existing lake template
* How to configure the various aspects of your retroactive run
* How to request execution of your retroactive run
* How to view and download results

Participants are encouraged to go over the tutorial on their own personal computers during the tutorial, in particular if they plan to do this for their research in the future. Participants may also follow along the screen-shared tutorial on Zoom if they do not run it themselves.

# Tutorial Goal

We want to run FLARE forecast for FCRE, but instead of forecasting into the future, we want to run the forecast on the historical data in the past. The period we are considering is July 01, 2022 to Jul 30, 2022 (30 days), and we want to do a cold start. It means that we do not have any prior forecast as the starting point or warm start. In a cold start, the forecast outputs for the first days are considered inaccurate. So, in order to have accurate results for the period, we should start the forecast a few days earlier, and ignore the first forecast results in our analysis. We plan to start the forecasts 5 days eralier on Jun 26, 2022 and run it for 35 days.

# Pre-requisites and installation

To follow this tutorial, you will need:

* A GitHub account
* An S3 client installed on your desktop to browse storage buckets

To help ensure a smooth tutorial, please go over these steps before the workshop:

## GitHub account

If you don't already have one, please create a GitHub account, and come to the tutorial already signed in to GitHub.

## Installing Rclone

Rclone is a free S3 client that works with multiple cloud storage technologies - including MinIO, which we use in CIBR. It is supported for Windows, macOS, and Linux. You need to install this software in your computer, so you can browse CIBR S3 buckets.

Please [follow the steps in this page to install and configure Rclone](Manage-Files-on-S3-Using-Rclone).

**Note:** For this tutorial, you only need read-only access - so it is ok to leave the `access_key_id` and `secret_access_key` blank when configuring Rclone.

If you have any difficulties installing/configuring Rclone, please contact Vahid prior to the tutorial.

# Workflow overview for retroactive forecasts

Before we go into the hands-on activities, let's overview the process. The main pieces of the puzzle here are:

## R code: GitHub repo
The forecast code to be executed is defined and stored in your own GitHub repository, and uses the FLARE library. So, you are free to customize the code to suit your needs, as per FLARE guidelines. Currently, we assume that you will use GLM as the model - but this can be expanded upon later.

## Lake configuration: GitHub repo
The configuration for your lake is also stored in the GitHub repo that has your code, in YAML format. The main configuration file is found in `configuration/default/configure_run.yml` - in this file you can specify values such as `sim_name`, `start_datetime` and `forecast_start_datetime` as per your needs. More on this later.

## Data: S3 buckets
The retroactive runs will read and write data from cloud storage using S3 buckets - a standard used in many cloud projects. CIBR has an S3 service running at Virginia Tech and based on the MinIO software; this is where your retroactive run outputs will be stored. The S3 bucket is readable by anyone - without the need for credentials. However, writes to the bucket need a valid credential. For retroactive runs, you will submit through someone who has already a credential managed on behalf of CIBR, so you will not need your own separate. However, if you do want to write directly to the bucket from your desktop for your use case, you will need credentials - let us know, and we'll work with you.

## Retroactive batch job configuration: JSON file
In addition to the YAML lake configuration described above - which would work even if you were to run a single forecast - you need to specify a retroactive job configuration file. This JSON-formatted file is intended to provide information about a _batch_ of runs.

## Retroactive batch job execution: Jetstream 2
The actual batch jobs (i.e. where FLARE, GLM run) currently take place in Jetstream 2, an NSF cloud resource we have access to. However, as an end user, you will not need to worry about this module - someone from the CIBR CI team will run the batch job on your behalf.

# Activity 1: prerequisite double-check

Do you have your GitHub account ready? Have you been able to install and configure Rclone successfully? Time to check on everyone to make sure you have your setup ready to move forward.

# Activity 2: browsing FLARE S3 storage

The s3flare S3 storage has several buckets where data for different lakes/forecast runs are stored. Here is an overview of the important buckets you need to be aware of:

## drivers

This bucket stores NOAA forecasts and inflow/outflow data, .csv (output of FLARE function 2). Example:

```bash
$ rclone lsf s3flare:drivers/noaa
NOAAGEFS_1hr/
NOAAGEFS_1hr-debias/
NOAAGEFS_1hr_stacked/
NOAAGEFS_1hr_stacked_average/
NOAAGEFS_6hr/
NOAAGEFS_6hr_stacked/
NOAAGEFS_raw/
```

```bash
$ rclone lsf s3flare:drivers/inflow
FLOWS-NOAAGEFS-AR1/
```

## targets
This bucket stores processed data from NOAA and observational data, both .csv and NetCDF (output of FLARE function 1). Example:

```bash
$ rclone lsf s3flare:targets/[lakecode]
fcre-targets-inflow.csv
fcre-targets-insitu.csv
observed-met_fcre.nc
```

## restart
This bucket stores YAML run configuration files ("configure_run.yml") for the next runs. There is a folder per "sim_name". (output of FLARE function 3). Example:

```bash
$ rclone lsf s3flare:restart/[lakecode]/[sim_name]
configure_run.yml
```

## forecasts
This bucket stores forecast outputs (output of FLARE function 3). Each file name (XML, NetCDF) has prefix [lakecode]_[sim_name]. Example:

```bash
$ rclone lsf s3flare:forecasts/[lakecode]
fcre_js2_H_2022_03_10_2022_05_10_F_0_20220518T014355.nc
fcre_js2_H_2022_03_10_2022_05_10_F_0_20220518T014355.xml
fcre_test4_H_2021_07_01_2021_09_01_F_0_20220114T182816.nc
fcre_test4_H_2021_07_01_2021_09_01_F_0_20220114T182816.xml
fcre_test5_H_2021_07_01_2021_09_01_F_0_20220114T010805.nc
fcre_test5_H_2021_07_01_2021_09_01_F_0_20220114T010805.xml
```

## analysis

This bucket stores graphical outputs of forecasts (output of FLARE function 4). Each file name (pdf) has prefix [lakecode]_[sim_name]. Example:

```bash
$ rclone lsf s3flare:analysis/[lakecode]
fcre_js2_H_2022_03_10_2022_05_10_F_0_20220518T014355.pdf
fcre_test4_H_2021_07_01_2021_09_01_F_0_20220114T182816.pdf
fcre_test5_H_2021_07_01_2021_09_01_F_0_20220114T010805.pdf
```

# Activity 3: editing FLARE configuration files in your lake repo
  
First step, you'll create a forecast code repository based on FCRE forecast:

1- Go to the following template which is based on FCRE-forecast-code:

https://github.com/FLARE-forecast/LAKE-forecast-code/

2- Click the green "Use this template" button on the top right corner of the page.

3- Choose a "Repository name" such as "TEST-forecast-code".

4- Click the green "Create repository from template" button on the bottom of the page.

In a real usage scenario, you would now work on the code to customize FLARE for your own lake. This can take time, and given that we have limited time for this tutorial, we will use FCRE forecast code without changes as an example. We will thus simply edit the configuration file to configure this tutorial run:

1- Edit the following file in your newly created repository by browsing to `configuration` then `default`:
  - `configuration/default/configure_run.yml`

2- Apply the changes to the `configure_run.yml` by following the template below - the line you need to worry about is highlighted:

```yaml
restart_file: .na
start_datetime: 2022-06-25 00:00:00
end_datetime: .na
forecast_start_datetime: 2022-06-26 00:00:00
forecast_horizon: 16.0
sim_name: tutorial_<yourname> # Update this line
configure_flare: configure_flare.yml
configure_obs: observation_processing.yml
use_s3: TRUE
```

**restart_file:** indicates the name of the latest forecast file if it is warm start. For a cold start, it is `.na`.

**start_datetime:** indicates historical data assimilation start date and time. For this tutorial, we set it to `2022-06-25 00:00:00`. It is usually one day before "forecast_start_datetime".

**end_datetime:** indicates historical data assilmilation end date and time.

**forecast_start_datetime:** indicates forecast start date and time. For this tutorial, we set it to `2022-06-26 00:00:00`. For a cold start, it should be at least a few days before the actual date you are considering for the forecast since the first few days' outputs are not accurate.

**forecast_horizon:** indicates the length of time in days into the future for which forecasts are to be generated. Since FCRE forecasts currently uses NOAA 16-day forecasts as one of the drivers, for this tutorial, we set it to `16.0`.

**sim_name:** indicates the simulation name which is being used to distinguish different forecast runs. The forecast outputs are categorized in different directories based on this key. For this tutorial, we set it to `tutorial_<yourname>`. This customizes the run so for you, separates your files from other people's files, and enables you to check on the results later.

**configure_flare:** indicates the name of the main configuration file.

**configure_obs:** indicates the name of the observation processing configuration file.

**use_s3:** identifies whether we want to use S3 storage to store forecast outputs or not. If set to `FALSE`, the outputs will be stored locally, otherwise, when set to `TRUE`, all the outputs will be transferred to the S3 storage.

The above configuration is set for running the forecasts starting June 26, 2022.

This is a configuration for "cold-starting", with no forecast history (`restart_file: .na`). So keep in mind that the forecast outputs and results for the first few days, let's say 5 days, are not accurate and should be ignored. When you configure your forecast retroactive runs, keep in mind to plan for this "ramp-up" period as you configure the YAML file.

# Activity 4: creating and submitting JSON configuration file
With your code repository set up and YAML configuration file created as per activity 3, the next step is to create a JSON file that is used to provide the necessary information for the retroactive batch run. Here is the template of the JSON file for a retroactive run:
```json
{
  "forecast_code_repo": "https://github.com/Yun-Jung/TEST-forecast-code",
  "forecast_code_branch": "main",
  "config_set": "default",
  "function": "0",
  "configure_run": "configure_run.yml",
  "use_https": "TRUE",
  "aws_default_region": "s3",
  "aws_s3_endpoint": "flare-forecast.org",
  "number_of_runs": 35
}
```
### Variables Explanation
* "forecast_code_repo" is the full URL link to your GitHub repository which you have created and configured earlier in activity 3.
* "forecast_code_branch" is the variable which points to the branch of your GitHub repository (typically main, but you can configure a different branch if needed).
* "configure_run" is the name of the YAML configuration file you edited in your GitHub repo in activity 4, i.e. where dates were defined.
* "config_set", "function" specify the FLARE config_set and which FLARE function to run (we'll use 'default' and '0' for this tutorial).
* "use_https", "aws_default_region", and "aws_s3_endpoint" specify S3-related configuration: whether to use https, which S3 region to use, and which S3 server to connect. For most (if not all) CIBR runs, these will all be "TRUE", "s3", and "flare-forecast.org", so you are unlikely to ever need to change these.
* "number_of_runs" is the variable which set the period of observed days. For example, if "number_of_runs" is set to "35" and the "forecast_start_datetime" is "2022/06/26 00:00:00" (defined in the "configure_flare.yml" as per the previous activity), there were results in 35 days in the S3 buckets.

After setting up these variables, the file needs to be saved as `tutorial_<yourname>.json`.
To start the retroactive run, send the JSON file to Yun-Jung (y.ku@ufl.edu). 
She will dispatch the run of a batch of 35 days as configured, using code from your repo with variables that you set up.
You will be able to check the results in the S3 bucket as the runs complete, as per activity 2. Files associated with you your runs will have `tutorial_<yourname>` in the name.

# Q&A time

# Workflow overview for daily forecasts

While in this tutorial we've gone through setting up a batch of retroactive forecast runs, the approach is fairly similar to run a daily operational forecast. The key difference is that, in retroactive batch mode, forecasts run “back-to-back” on data from the past, where as in operational mode, forecasts run daily at a configurable time (or multiple times daily, if needed) on current data. In summary, to setup a daily forecast:

* You need your own code repository, configured for your lake, as we did in this tutorial.
* You need to ensure drivers and observational data are available for your lake (from S3 or GitHub).
* You need to produce and submit a pull request for a JSON configuration file describing your forecast.

[This page provides a detailed how-to if you are interested](https://github.com/FLARE-forecast/deployed-forecasts/blob/master/README.md)

# Activity 5: checking your outputs

Use the Rclone commands from activity 2 to inspect outputs in the S3 bucket. Your forecast/analysis outputs will have `tutorial_<yourname>`.

Note that it's likely your retroactive jobs are still running while you do this, and you may only see outputs for a few days in the meantime.

# Discussion: improvements, next steps

Some ideas/questions for discussion:

* Will you have use for this system in your research?
* What do you think could be improved in this process to fit your research workflow?
* What were the parts of this process that were the least clear to you?
* Do you think users outside the CIBR project would be able to use this?