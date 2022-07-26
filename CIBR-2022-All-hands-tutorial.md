# Introduction and overview of tutorial

During the 2022 CIBR virtual all-hands meeting, participants will go over a tutorial on how to configure and request execution of a set of retroactive forecast runs using FLARE. This is a capability that can facilitate CIBR researchers to execute batches of retroactive forecast runs. For example, let's say a researcher comes up with a new enhancement to the GLM model or FLARE data assimilation and would like to compare how it fares against a current approach using FCR as a case study. They will be able to use what they learned in this tutorial to run retroactive forecasts for FCR using current and new approach, say, for the entirety of 2020.

In this tutorial, you will learn:

* An overview of the workflow, in particular: where the code, data, and configuration reside, and how the execution takes place using containers
* How to navigate data used by the workflow stored in cloud storage - S3 buckets
* How to create your own "starter lake" by forking from an existing lake
* How to configure the various aspects of your retroactive run
* How to request execution of your retroactive run
* How to view and download results

Participants are encouraged to go over the tutorial on their own desktops during the tutorial, in particular if they plan to do this for their research in the future. Participants may also follow along the screen-shared tutorial on Zoom if they do not run it themselves.

# Pre-requisites and installation

To follow this tutorial, you will need:

* A GitHub account
* A lake properly configured with the FLARE code and configuration in one of your repositories
* An S3 client installed on your desktop to browse storage buckets

To help ensure a smooth tutorial, please go over these steps before the workshop:

## Creating your own FLARE lake from a template

If you don't already have one, please create a GitHub account. 

* [Follow the instructions from the FLARE template README](https://github.com/FLARE-forecast/LAKE-forecast-code#readme) to create your own lake (based off FCR) in your own GitHub repository

## Installing Rclone

Rclone is a free S3 client that works with multiple cloud storage technologies - including MinIO, which we use in CIBR. It is supported for Windows, macOS, and Linux. You need to install this software in your computer, so you can browse CIBR S3 buckets.

Please [follow the steps in this page to install and configure Rclone](Manage-Files-on-S3-Using-Rclone). Note: for this tutorial, you only need read-only access - so it is ok to leave the `access_key_id` and `secret_access_key` blank when configuring Rclone.

# Workflow overview for retroactive forecasts

Before we go into the hands-on activities, let's overview the process. The main pieces of the puzzle here are:

## R code: GitHub repo
The forecast code to be executed is defined and stored in your own GitHub repository, and uses the FLARE library. So, you are free to customize the code to suit your needs, as per FLARE guidelines. Currently, we assume that you will use GLM as the model - but this can be expanded upon later

## Lake configuration: GitHub repo
The configuration for your lake is also stored in the GitHub repo that has your code, in YAML format. The main configuration file is found in `configuration/default/configure_run.yml` - in this file you can specify values such as `sim_name`, `start_datetime` and `forecast_start_datetime` as per your needs. More on this later

## Data: S3 buckets
The retroactive runs will read and write data from cloud storage using S3 buckets - a standard used in many cloud projects. CIBR has an S3 service running at Virginia Tech and based on the MinIO software; this is where your retroactive run outputs will be stored. The S3 bucket is readable by anyone - without the need for credentials. However, writes to the bucket need a valid credential. For retroactive runs, you will submit through someone who has already a credential managed on behalf of CIBR, so you will not need your own separate. However, if you do want to write directly to the bucket from your desktop for your use case, you will need credentials - let us know, and we'll work with you.

## Retroactive batch job configuration: JSON file
In addition to the YAML lake configuration described above - which would work even if you were to run a single forecast - you need to specify a job configuration file. This JSON-formatted file is intended to provide information about a _batch_ of runs 

## Retroactive batch job execution: Jetstream 2
The actual batch jobs (i.e. where FLARE, GLM run) currently take place in Jetstream 2, an NSF cloud resource we have access to. However, as an end user, you will not need to worry about this module - someone from the CIBR CI team will run the batch job on your behalf

# Activity 1: prerequisite double-check

# Activity 2: browsing FLARE S3 bucket

The s3flare S3 bucket has several folders where data for different lakes/forecast runs are stored. Here is an overview of the important folders you need to be aware of:

## drivers

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

Outflow data, .csv (output of FLARE function 2). Example:

```bash
$ rclone lsf s3flare:drivers/inflow
FLOWS-NOAAGEFS-AR1/
```

## targets
Processed data from NOAA and observational data, both .csv and NetCDF (output of FLARE function 1). Example:

```bash
$ rclone lsf s3flare:targets/[lakecode]
fcre-targets-inflow.csv
fcre-targets-insitu.csv
observed-met_fcre.nc
```

## restart
This stores the YML run configuration file (configure_run.yml) for the next run. There is a folder per sim_name. (output of FLARE function 3). Example:

```bash
$ rclone lsf s3flare:restart/[lakecode]/[sim_name]
configure_run.yml
```

## forecasts
This stores the forecast outputs (output of FLARE function 3). Each file name (xml, NetCDF) has prefix [lakecode]_[sim_name]. Example:

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
Graphical outputs of forecasts (output of FLARE function 4). Each file name (pdf) has prefix [lakecode]_[sim_name]. Example:

```bash
$ rclone lsf s3flare:analysis/[lakecode]
fcre_js2_H_2022_03_10_2022_05_10_F_0_20220518T014355.pdf
fcre_test4_H_2021_07_01_2021_09_01_F_0_20220114T182816.pdf
fcre_test5_H_2021_07_01_2021_09_01_F_0_20220114T010805.pdf
```

# Activity 3: editing FLARE configuration files in your lake repo
  
For creating a new lake forecast code based on FCRE forecast, follow these steps:

1- Go to the following template which is based on FCRE-forecast-code:

https://github.com/FLARE-forecast/LAKE-forecast-code/

1- Click the green "Use this template" button on the top right corner of the page.

2- Choose a "Repository name" such as "TEST-forecast-code".

3- Click the green "Create repository from template" button on the bottom of the page.

4- Edit the following files in your newly created repository:
  - `configuration/default/configure_flare.yml`

5- Apply the changes to the `configure_flare.yml` as the following sample:

```yaml
restart_file: .na
start_datetime: 2022-06-01 00:00:00 # Update this line
end_datetime: .na
forecast_start_datetime: 2022-06-30 00:00:00 # Update this line
forecast_horizon: 16.0 # Update this line
sim_name: tutorial_<yourname> # Update this line
configure_flare: configure_flare.yml
configure_obs: observation_processing.yml
use_s3: TRUE
```

The above configuration is for running the forecasts for 30 days starting May 01, 2022 and `sim_name: tutorial_<yourname>` customizes that for you and enables you to check on the results later. Since it is cold starting with no forecast history (`restart_file: .na`), the forecast outputs and results for the first few days, let's say 5 days, are not accurate and should be ignored.  

# Activity 4: creating and submitting JSON configuration file

HERE - Yun-Jung to show JSON template, explain key/values, and explain how to send it to you

# Q&A time

# Workflow overview for daily forecasts

While in this tutorial we've gone through setting up a batch of retroactive forecast runs, the approach is fairly similar to run a daily operational forecast. The key difference is that, in retroactive batch mode, forecasts run “back-to-back” on data from the past, where as in operational mode, forecasts run daily at a configurable time (or multiple times daily, if needed) on current data. In summary, to setup a daily forecast:

* You need your own code repository, configured for your lake, as we did in this tutorial
* You need to ensure drivers and observational data are available for your lake (from S3 or GitHub)
* You need to produce and submit a pull request for a JSON configuration file describing your forecast

[This page provides a detailed how-to if you are interested](https://github.com/FLARE-forecast/deployed-forecasts/blob/master/README.md)

# Activity 5: checking your outputs

# Discussion: improvements, next steps