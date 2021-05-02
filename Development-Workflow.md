## FLARE Development Workflow

The steps in the development of code and its packaging and deployment in containers for FLARE are outlined below

### Repositories

* Code for various FLARE modules resides in: https://github.com/FLARE-forecast/FLARE

Note: FLARE code has been forked from https://github.com/CareyLabVT/FLARE in April 2020; development continues in that repository only for minor updates and bug fixes as needed for the SCC/Smart Reservoirs project

* Dockerfiles, scripts and configuration files for FLARE containers reside in: https://github.com/FLARE-forecast/FLARE-containers

Note: each FLARE container has its own branch in the FLARE-containers repository

* Datasets curated for development and testing reside in: https://github.com/FLARE-forecast/test-data

* FLARE containers are published on Dockerhub in: https://hub.docker.com/r/flareforecast

* FLARE containers that run R will be built upon Rocker with Rstudio and tidyverse: https://hub.docker.com/r/rocker/tidyverse 

### Container naming conventions

* FLARE containers have a prefix "flare-"; for instance, "flare-download-noaa".

* FLARE containers under development/testing have a suffix "-dev"; for instance, "flare-download-noaa-dev".

### Code development and testing

* During development, use the "dev" branch for FLARE code
* Development of R code can be done from your own environment - e.g. Rstudio on your own desktop
* To match the approach taken by our containers, ensure that all inputs/outputs should be retrieved from/pushed to GitHub test repositories - i.e., your code should work in a stateless fashion
* Once you have gone through sufficient development/testing on your local environment, and would like to test on a FLARE container, commit your changes to the "dev" branch of the FLARE code
** This triggers an automated build process to generate and publish a container image with -dev suffix 
* Download and deploy the -dev container to your own computer for testing
** We recommend a Linux environment to deploy your -dev container; this can be accomplished by running within a Linux Ubuntu virtual machine with VMware or VirtualBox, to mimic a production deployment environment
** Set your container configuration files to run in interactive mode, and to ensure that all inputs/outputs should be retrieved from/pushed to GitHub test repositories
* Once a -dev container has been successfully tested and you wish to have the changes committed to a production deployment
** Issue a pull request for the FLARE "master" branch
** The pull request needs to be reviewed by at least one other developer before committed
** Commits to the master branch will trigger automated builds of production FLARE containers, which are then deployed in future forecast cycles by OpenWhisk

## Dealing with Shared Content among Multiple Runs

### Running Containers in Manual Mode

It uses a shared volume on the host machine as the working directory for shared content among the containers and also the files needed to be preserved for the next runs of the containers. "Manual Mode" is the default mode for running the containers. 

### Running Containers in OpenWhisk Mode

It uses a GitLab container for shared content among the containers and also the files needed to be preserved for the next runs of the containers. The working directory is stored in compressed format on a daily basis and can be restored when needed. To run a container in OpenWhisk mode, you can use `-o` or `--openwhisk` parameter when executing `flare-host.sh` or `flare-container.sh`.

## Branching Structure on GitHub

### Main and Development Branches

Each container has two branches under `https://github.com/FLARE-forecast/FLARE-containers/` repo:
- **{$CONTAINER} Main Branch:** Keeps the fully tested *production* code for the container. For instance:  
https://github.com/FLARE-forecast/FLARE-containers/tree/flare-download-noaa

- **{$CONTAINER}-dev Development Branch:** Keeps the on-going *development* code for the container. It may be buggy and not ready for production. For instance:  
https://github.com/FLARE-forecast/FLARE-containers/tree/flare-download-noaa-dev

### Commons Branch

Common files among containers are stored under the "commons" branch:  
https://github.com/FLARE-forecast/FLARE-containers/tree/commons

## Publishing Containers on DockerHub

The *production* container image based on the main branch gets published on DockerHub under `https://hub.docker.com/repository/docker/flareforecast/`. For instance:  
https://hub.docker.com/repository/docker/flareforecast/flare-download-noaa

The *development* container image based on the dev branch doesn't get published on DockerHub. It must be manually built from the `Dockerfile`.