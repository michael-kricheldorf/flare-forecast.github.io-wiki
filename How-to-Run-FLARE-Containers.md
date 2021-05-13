**NOTE:** For running the commands, set `FLARE_CONTAINER_NAME` environment variable to the container name you are willing to run:
- flare-download-noaa
- flare-process-noaa
- flare-download-observations
- flare-process-observations
- flare-generate-forecast
- flare-visualize

For instance:

```bash
$ export FLARE_CONTAINER_NAME=flare-download-noaa
```

Make sure the environment variable has been set properly by printing that in the terminal

```bash
$ echo $FLARE_CONTAINER_NAME
```

The output should be the name of the variable you have already set:

```bash
flare-download-noaa
```

If you want to use a specific version, set `FLARE_VERSION` environment variable. For instance:

```bash
$ export FLARE_VERSION=21.01.4
```

If no version is specified, the default `latest` tag will be used.


# Prerequisite

Make sure you can [access GitHub with SSH], if you are using a private repository. (https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/connecting-to-github-with-ssh).


# Pull container image

```bash
$ docker pull flareforecast/$FLARE_CONTAINER_NAME:$([[ -z "$FLARE_VERSION" ]] && echo 'latest' || echo "$FLARE_VERSION")
```


# Install Container on Host

Run the `flare-install.sh` script to copy the required files into `~/flare-host` directory. 

```bash
$ source <(wget -O - https://raw.githubusercontent.com/FLARE-forecast/FLARE-containers/$([[ -z "$FLARE_VERSION" ]] && echo 'latest' || echo "$FLARE_VERSION")/commons/flare-install.sh | /usr/bin/env bash -s $FLARE_CONTAINER_NAME $([[ -z "$FLARE_VERSION" ]] && echo 'latest' || echo "$FLARE_VERSION"))
```


# Update Configurations

Change the `flare-config.yml` file and update your GitHub user-name and user-email and any other settings you want:

```bash
$ vi ~/flare-host/shared/$FLARE_CONTAINER_NAME/flare-config.yml
```


# Run FLARE Host Script

Run the `flare-host.sh` script:

```bash
$ ~/flare-host/$FLARE_CONTAINER_NAME/flare-host.sh
```

It runs the container and the container runs the `flare-container.sh` script which does the job.


# Source Code

https://github.com/FLARE-forecast/FLARE-containers


# Cleanup

When you are done, unset the `FLARE_CONTAINER_NAME` and `FLARE_VERSION` environment variables:

```bash
$ unset FLARE_CONTAINER_NAME
$ unset FLARE_VERSION
```

# First Run Configuration

## flare-generate-forecast Container

To run for the first time, you should run the `flare-host.sh` script once. It creates the `run_configuration.yml` file at `/opt/flare/shared/flare-generate-forecast/forecast/configuration_files/run_configuration.yml`. Then edit this file from the host as follows: (Suppose the first date for whom you have NOAA forecasts is `2021-02-23`.)

```bash
$ sudo vi /root/flare/shared/flare-generate-forecast/forecast/configuration_files/run_configuration.yml
```

```yaml
restart_file: .na
start_day_local: '2021-02-22'
start_time_local: '07:00:00'
end_day_local: .na
forecast_start_day_local: '2021-02-23'
forecast_horizon: 16.0
forecast_project_id: wrr_runs
sim_name: wrr_runs
forecast_sss_on: no
execute_location: /home/user/flare/shared/flare-generate-forecast/working_directory
```