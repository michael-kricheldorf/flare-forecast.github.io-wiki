The configuration file for FLARE containers uses the YAML format. Multiple containers that run on the same host are configured with the same YAML file; each container's configuration is preceded by the container's name, as in the example below.

**Note: this is an example of the structure of the YAML file that may not reflect the latest configuration of a particular container. Please refer to each container's documentation in the FLARE wiki for the most up-to-date configuration information**

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
  name: flare-process-noaa
  working-directory:
    pre-run-pull: FALSE
    post-run-push: FALSE
    git:
      remote:
        server: 192.168.20.20
        port:
        repository: FLARE-forecast/test-data
        branch: master
        directory:
## Container Paths
flare_path: /opt/flare
flare_shared_path: /opt/flare/shared
read_from_path: /opt/flare/shared/flare-download-noaa
## Parameters
downscale: TRUE
run_parallel: TRUE
overwrite: FALSE
num_cores: 8
forecast_date: .na
forecast_time: .na
lake_name_code: fcre
lake_name: Falling Creek Reservoir
lake_latitude: 37.307   #Degrees North
lake_longitude: 79.837  #Degrees West
```