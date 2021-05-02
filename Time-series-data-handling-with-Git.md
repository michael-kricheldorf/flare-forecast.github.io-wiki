# Introduction

FLARE builds upon [Git services to handle time-series data from sensors](Role-of-Git-in-FLARE). This document overviews the workflow for handling time-series data in FLARE using Git.

# Overview and terminology

## Systems

These are the main users/systems that interact with Git data in FLARE:

* Data Logger: responsible for acquiring data from sensors; deployed in the field
* Gateway: responsible for staging data from the logger, and transferring data over the Internet; also deployed in the field/edge, and connected to the Data Logger via a local interface (e.g. Ethernet)
* Git Server: responsible for storing data in the cloud, and exposing it via the Git protocol; may be deployed in a public cloud service (e.g. GitHub), or in a private server (e.g. Gitlab Docker container)
* Git User Client: responsible for manually editing data when needed (e.g. gap-filling, manual QA/QC) and issuing pull requests; typically a laptop or desktop computer
* Git Management Client: responsible for management of Git repository data files (e.g. handling large files/repos); typically a laptop/desktop, or a cloud instance
* Long-term Archive: responsible for long-term, persistent storage of processed time-series data with associated meta-data; for example, [EDI](https://environmentaldatainitiative.org/edi/)

## Git storage

These are Git repositories, branches and files associated with Git data in FLARE:

* lake-data-raw: a Git repository storing raw time-series data for a lake
* lake-data-processed: a Git repository storing processed time-series data for a lake
* lake-site-data: a branch of a Git repository storing data for a particular sensor site (e.g. buoy, catwalk, metstation, inflow)


## Files

* lake-timeseriesname-raw_id.csv: a file containing raw time series data from a given sensor
* lake-timeseriesname-processed_id.csv: a file containing processed time series data from a given sensor
* Each line/row of the .csv files has a monotonically-increasing timestamp: i.e., if line=x has a timestamp Tx, any line y<x must have a timestamp Ty <= Tx
* The identifier "id" is also monotonically increasing: i.e., a file with id=i must have rows with timestamps Ti that are greater than or equal to all timestamps in all rows of a file with id=j, for all j<i


# Configuration

## At Git server

The Git server needs to be configured by creating appropriate repositories for a lake, and adding SSH keys to provide access to authorized users. This involves steps particular to a given Git server, e.g. creating a GitHub account, deploying a GitLab container, etc.

## At Data Logger and Gateway

The Data Logger and Gateway need to be configured such that:

* Raw data from the Data Logger can be uploaded to the Gateway (e.g. using Campbell Logger's FTP)
* Data from the Gateway can be pushed to the Git Server (e.g. via SSH, with proper credentials)

## At Git User Client and Management Client

These need to be configured with proper credentials and Git client software to access the Git Server. They may also be configured with proper credentials to access the Long-term Archive

# Workflow overview

## At Gateway

The workflow at the Gateway consists of:

* An automated process to retrieve raw data from the Data Logger, and store it locally as lake-timeseriesname-raw_id.csv 
* Note that raw time-series data from the Data Logger may overwrite the lake-timeseriesname-raw_id.csv file stored locally in the gateway
* A (manual) process to generate a new id and reconfigure Data Logger and Gateway, when a data logger's storage capacity is near exhausted
* An automated process to selectively append new data from lake-timeseriesname-raw_id.csv into lake-timeseriesname-processed_id.csv, as explained below
* An automated process to pull/push data from the Git Server repositories for raw and processed data

For the most part, lake-timeseriesname-processed_id.csv holds the same data as collected lake-timeseriesname-raw_id.csv; however, there are instances when data needs to be overridden (e.g. a manual update of incorrect sensor data). To support this, the workflow is:

* Raw data from the logger goes into lake-timeseriesname-raw_maxidR.csv of the lake-data-raw repository, and is not further processed, where maxidR is the largest id for raw data in the repo
* Processed data goes into lake-timeseriesname-processed_maxidP.csv of the lake-data-processed repository, where maxidP is the largest id for processed data in the repo
* At the Gateway, this is performed with a combination of Git pulls, an append process, and a Git push:
* The Gateway pulls lake-timeseriesname-processed_maxidP.csv from lake-data-processed
* The Gateway runs an append process that takes a subset of data from lake-timeseriesname-raw_maxidR.csv and appends to lake-timeseriesname-processed_maxidP.csv
* The Gateway pushes the appended lake-timeseriesname-processed_maxidP.csv to lake-data-processed
* The Gateway also pushed lake-timeseriesname-raw_maxidR.csv to lake-data-raw

The append process works as follows. The intuition is to check for all differences, but only append new raw data:

* generate a temporary diff_file = diff (lake-timeseriesname-raw_maxidR.csv, lake-timeseriesname-processed_maxidP.csv)
* Scan diff_file backwards (i.e. from end of file backwards to begin) line by line
* Compare the timestamp Tscandiff of line read from diff_file with the timestamp of last line of lake-timeseriesname-processed_maxidP.csv Tprocessedlast
* If Tscandiff <= Tprocessedlast, stop. Then, read diff_file line by line, starting from Tscandiff, and append each line from diff_file to lake-timeseriesname-processed_maxidP.csv

## At Git User Client

Occasional manual updates can be handled as follows:

* Pull latest data from lake-data-processed
* Edit time series data
* Generate a pull request

Archival can also be handled:

* Pull latest data from lake-data-processed
* Generate metadata
* Commit to Long-term Archive

## At Git Management Client

Occasional management operations can be performed to handle file/repo sizes:

* Pull latest data from lake-data-processed
* Create an empty lake-timeseriesname-processed_maxidP+1.csv file in lake-data-processed 
* Populate the file with the last Nentries from lake-timeseriesname-processed_maxidP.csv
* Ndays is a configurable value
* If needed, archive older i<maxidP files and remove from lake-data-processed repo







