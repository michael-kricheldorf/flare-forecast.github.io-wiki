**NOTE:** For running the commands, set `FLARE_CONTAINER_NAME` environment variable to the container name you are willing to run:
- flare-download-noaa (In Progress, Not Functional Yet)
- flare-process-noaa
- flare-download-observations
- flare-process-observations
- flare-generate-forecasts
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


# Prerequisite

Make sure you can [access GitHub with SSH](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/connecting-to-github-with-ssh).


# Build FLARE Image

The easiest way to build a new version of a container image is to modify the latest version, apply the new changes, and save it as a new image using `docker commit`.

## Build an Image from Scratch

### Clone Codebase

```bash
$ git clone git://github.com/FLARE-forecast/FLARE-containers
$ cd FLARE-containers/$FLARE_CONTAINER_NAME
```

### Add `flare_lake_examples` Submodule

```bash
$ git submodule add git@github.com:FLARE-forecast/flare_lake_examples.git
```

### Apply `flare_lake_examples` Submodule Modifications

#### File Diffs

[02_process_observations.R](https://www.diffchecker.com/fbVNm0B9)

[03_generate_forecast.R](https://www.diffchecker.com/qEkmZhvj)

[04_visualize.R](https://www.diffchecker.com/31XlWl4I)

[visualization/manager_plot.R](https://www.diffchecker.com/FhAymQoC)

### Build Image

```bash
$ docker build -t flareforecast/$FLARE_CONTAINER_NAME -f $FLARE_CONTAINER_NAME/Dockerfile .
```