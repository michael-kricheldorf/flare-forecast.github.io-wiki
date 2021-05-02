## Documentation for flare-visualize-dev container

### Overview of container functionality

This container is responsible for visualizing FLARE forecast output in graphs.

### Key software dependencies

### Trigger

This container is triggered after a successful run of `flare-generate-forecast-dev` container. 

### Inputs and outputs

**Input:** NetCDF Files

**Output:** Graphs in PDF Format

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
  name: flare-process-observations-dev
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

##########################
# Lake information
###########################

lake_name_code: fcre
lake_name: Falling Creek Reservoir
```

## [How to Run FLARE Container](https://github.com/FLARE-forecast/FLARE/wiki/How-to-Run-FLARE-Containers)