# Introduction

FLARE containers running in OpenWhisk mode use remote storage from a networked server. Each container is stateless, so it needs to "pull" data it needs at the beginning of execution, and "push" data it produces at the end. 

Typically, when there are no errors, containers run once a day for a forecast cycle; when there are errors, containers may run more than once a day. The exception is flare-download-noaa, which is scheduled to run multiple times a day.

Different containers operate on different data. Some containers depend on data from its own executions from previous days to execute with good performance. For example, flare-download-noaa uses files downloaded in previous days from the NOAA server, and flare-download-data uses git repository files fetched from its git repository from the previous day.

Overall, the way we deal with these requirements is as follows:

* Each container starts "blank", with an empty file system
* A container then needs to download ("pull") one or more working directories into its file system. At a minimum, it downloads its own directory from a previous day (or the same day, if it's restarted); some containers may need to download additional working directories produced by others
* The container then does its job (e.g. download more data for the day, generate a forecast, etc), at which point it needs to upload ("push") its working directory before it terminates

Because each working directory can be several layers deep with subdirectories, and because we want to package this neatly in a way that facilitates storage, the way we handle this is to archive/compress the entire working directory into a tar/gzip archive. Hence, "pulling" working directories amounts to downloading and expanding one (or more) .tgz files from remote storage, and "pushing" working directories amounts to compressing and uploading one .tgz file. This way, the server can use scp, and we can move to other storage technologies later (e.g. S3) should we want to.


# Naming scheme

The naming scheme for each working directory archive is as follows:

lakename_date_containername_workingdirectory.tgz

Where:

* lakename is the 4-letter code for a lake (e.g. fcre for Falling Creek Reservoir), and it is passed to the container via the configuration file
* date is today's date, in the format YYYYMMDD, e.g. 20210211 for Feb. 11 2021
* containername is the FLARE container name, e.g. flare-download-noaa
* workingdirectory is a constant name
* tgz is the gziped tar archive

One example of an archive produced by flare-download-noaa on Feb. 11 2021 for the fcre lake would be:

```
fcre_20210211_flare-download-noaa_workingdirectory.tgz
```

# Push/pull working directory logic

## Push 

For push, we always upload *only today's date for this container*. In other words, a container only produces one output working directory - each container may use working directories from multiple containers, but it only *creates* an archive for its own working directory. The working directory has the same name as the container, and is created in the container shared directory, typically `/root/flare/shared/<containername>`

For instance, a container such as flare-download-noaa, when it finishes its job and is ready to push, should push only today's data for itself, e.g.:

```
cd /root/flare/shared/
tar -czf fcre_20210211_flare-download-noaa_workingdirectory.tgz flare-download-noaa
scp fcre_20210211_flare-download-noaa_workingdirectory.tgz <user>@<server>:fcre_20210211_flare-download-noaa_workingdirectory.tgz
```

## Pull

Pulling is more complicated because:

* A container may need to pull from multiple sources
* A container needs to determine the right date to pull from (today, or earlier)

We need a scheme where a container can be configured to pull the latest working directories (for itself and any containers it depends on, starting from today and going backwards in time. 

Both the set of dependences and the backward-in-time look should be configurable (e.g. to a single day, yesterday, or to a longer timeframe, e.g. a couple of weeks). We can then perform this search, as in the following pseudocode:

```
cd /root/flare/shared
for each containername in ({set_of_dependences, from config file}) {
  downloaded = False
    for scandate in (today backwards {Ndays steps, from config file}) {
      if downloaded == False {
        if (scp server:lakename_scandate_containername_workingdirectory.tgz .) is successful:
          downloaded = True
          tar -xzf lakename_scandate_containername_workingdirectory.tgz
      }
    }
  }
  if downloaded == False {exit container with error - could not pull working directory}
}
```


# Dependences and what goes into the working directory of each container

## flare-download-noaa

### set_of_dependences to pull from:

The set is only this container:

* flare-download-noaa

### Ndays look back:

This can look back N=1 (yesterday)

### Working directory contents:

The working directory (inside flare-download-data) should have the following data for *only today and the past 4 days*:

* raw data downloaded from NOAA by the QueueDownloader
* NOAAGEFS_1hr data
* NOAAGEFS_6hr data

Important: Data older than 4 days should be removed before pushing, so that old data does not accumulate in the working directory


## flare-download-data

### set_of_dependences to pull from:

The set is only this container:

* flare-download-data

### Ndays look back:

This can look back N=1 (yesterday)

### Working directory contents:

The working directory has the data in git repos from the sensors; git manages updates here, so we don't need to delete anything



## flare-process-observations

### set_of_dependences to pull from:

The set of working directory dependencies to pull from is: 

* flare-download-data

### Ndays look back:

This can look back N=0 (just today)

### Working directory contents:

No need to delete anything for now; we may want to fine-tune later



## flare-generate-forecast

### set_of_dependences to pull from:

The set of working directory dependencies to pull from is: 

* flare-generate-forecast
* flare-download-noaa
* flare-process-observations

### Ndays look back:

This can look back N=1 (today and yesterday)

### Working directory contents:

No need to delete anything for now; we may want to fine-tune later



## flare-visualize

### set_of_dependences to pull from:

The set of working directory dependencies to pull from is: 

* flare-generate-forecast

### Ndays look back:

This can look back N=0 (just today)

### Working directory contents:

No need to delete anything for now; we may want to fine-tune later


# Removing old working directories

We can create a garbage collector container that is invoked daily and removes old data from the server. This will be a little tricky with scp, as there's no delete operation, but we can cross that bridge later.
