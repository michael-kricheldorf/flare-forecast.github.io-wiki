# Introduction and overview of tutorial

During the 2022 SIBR virtual all-hands meeting, participants will go over a tutorial on how to configure and request execution of a set of retroactive forecast runs using FLARE. This is a capability that can facilitate CIBR researchers to execute batches of retroactive forecast runs. For example, let's say a researcher comes up with a new enhancement to the GLM model or FLARE data assimilation and would like to compare how it fares against a current approach using FCR as a case study. They will be able to use what they learned in this tutorial to run retroactive forecasts for FCR using current and new approach, say, for the entirety of 2020.

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

## Forking your own FLARE test lake

If you don't already have one, please create a GitHub account. 

Then, use the GitHub "fork" to create a fork of the FCR lake in your own account:

* Browse to the FCR forecast code repository, [https://github.com/FLARE-forecast/FCRE-forecast-code](https://github.com/FLARE-forecast/FCRE-forecast-code)
* Locate the "Fork" icon towards top right of your screen; click on the down arrow and select "Create a new fork"


## Installing RClone

RClone is a free S3 client that works with multiple cloud storage technologies - including Minio, which we use in CIBR. It is supported for Windows, MacOS, and Linux. You need to install this software in your computer so you can browse CIBR S3 buckets.

Please [follow the steps in this page to install and configure RClone](Manage-Files-on-S3-Using-RClone). Note: for this tutorial, you only need read-only access - so it is ok to leave the access_key_id and secret_access_key blank when configuring RClone.

# Workflow overview for retroactive forecasts

Before we go into the hands-on activities, let's overview the process. The main pieces of the puzzle here are:

## R code: GitHub repo
The forecast code to be executed is defined and stored in your own GitHub repository, and uses the FLARE library. So, you are free to customize the code to suit your needs, as per FLARE guidelines. Currently, we assume that you will use GLM as the model - but this can be expanded upon later

## Lake configuration: GitHub repo
The configuration for your lake is also stored in GitHub repo that has your code, in YAML format. The main configuration file is found in configuration/default/configure_run.yml - in this file you can specify values such as sim_name, start_datetime and forecast_start_datetime as per your needs. More on this later

## Data: S3 buckets
The retroactive runs will read and write data from cloud storage using S3 buckets - a standard used in many cloud projects. CIBR has an S3 service running at Virginia Tech and based on the Minio software; this is where your retroactive run outputs will be stored. The S3 bucket is readable by anyone - without the need for credentials. However, writes to the bucket need a valid credential. For retroactive runs, you will submit through someone who has already a credential managed on behalf of CIBR, so you will not need your own separate. However, if you do want to write directly to the bucket from your desktop for your use case, you will need credentials - let us know and we'll work with you.

## Retroactive batch job configuration: JSON file
In addition to the YAML lake configuration described above - which would work even if you were to run a single forecast - you need to specify a job configuration file. This JSON-formatted file is intended to provide information about a _batch_ of runs 

## Retroactive batch job execution: Jetstream 2
The actual batch jobs (i.e. where FLARE, GLM run) currently take place in Jetstream 2, an NSF cloud resource we have access to. However, as an end user, you will not need to worry about this module - someone from the CIBR CI team will run the batch job on your behalf

# Activity 1: pre-requisite double-check

# Activity 2: browsing FLARE S3 bucket

# Activity 3: editing FLARE configuration files in your lake repo

# Activity 4: creating and submitting JSON configuration file

# Q&A time

# Workflow overview for daily forecasts

# Activity 5: checking your outputs

# Discussion: improvements, next steps