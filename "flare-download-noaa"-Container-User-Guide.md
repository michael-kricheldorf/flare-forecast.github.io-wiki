## Documentation for flare-download-noaa-dev container

### Overview of container functionality

This container is responsible for downloading meteorological data from NOAA, on a daily basis, for one or more geo-coordinates.


### Key software dependencies

* System Packages: R
* R Packages: rNOMADS, RCurl, stringr, yaml


### Trigger

This container is triggered on a daily timer.


### Inputs and outputs

**Input:** Name, Latitude, and Longitude of a Site

**Output:** NOAA Forecast for the Site


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
  name: flare-download-noaa-dev
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
  site:
    name: fcre
    latitude: 37.27
    longitude: 79.9
```


## [How to Run FLARE Container](https://github.com/FLARE-forecast/FLARE/wiki/How-to-Run-FLARE-Containers)