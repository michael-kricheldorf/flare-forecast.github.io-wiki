## Documentation for flare-download-data-dev container

### Overview of container functionality

This container is responsible for downloading sensor data for a specific location required for running the forecasts.


### Key software dependencies


### Trigger

This container is triggered on a daily timer 


### Inputs and outputs

**Input:** Name of a Site, Paths to the Sensor Data of the Site

**Output:** Downloaded Sensor Data of the Site


### Error logs


### Configuration file

```yaml
### Configuration File for CIBR-FLARE Project
## General Settings
git:
  remote:
    server: github.com
    branch: master
    ssh-key-private:
    user-name:
    user-email:
## Container Settings
container:
  name: flare-download-data-dev
  working-directory:
    pre-run-pull: FALSE
    post-run-push: FALSE
    git:
      remote:
        server: 192.168.20.20
        port:
        repository: FLARE-forecast/test-data
        branch: master
        directory: fcre
        ssh-key-private:
        user-name:
        user-email:
## Parameters
lake_name_code: fcre
lake_name: Falling Creek Reservoir
noaa_location: flare-download-noaa-dev/fcre
realtime_insitu_location:
  git:
    remote:
      server: github.com
      repository: FLARE-forecast/FCRE-data
      branch: fcre-catwalk-data
      directory:
realtime_met_station_location:
  git:
    remote:
      server: github.com
      repository: FLARE-forecast/FCRE-data
      branch: fcre-metstation-data
      directory:
manual_data_location:
  git:
    remote:
      server: github.com
      repository: FLARE-forecast/FCRE-data
      branch: fcre-manual-data
      directory:
  manual-downloads:
  - url: https://portal.edirepository.org/nis/dataviewer?packageid=edi.389.4&entityid=c1db8816742823eba86696b29f106d0f
    file-name: Met_final_2015_2019.csv
realtime_inflow_data_location:
  git:
    remote:
      server: github.com
      repository: FLARE-forecast/FCRE-data
      branch: fcre-weir-data
      directory:
```


## [How to Run FLARE Container](https://github.com/FLARE-forecast/FLARE/wiki/How-to-Run-FLARE-Containers)