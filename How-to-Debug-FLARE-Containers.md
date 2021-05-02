# Access Container Bash for Debugging

```
$ docker run -v /opt/flare/shared/:/root/flare/shared/ -it flareforecast/$FLARE_CONTAINER_NAME /bin/bash
```

Run `flare-container.sh` script in debug mode from the bash inside the container and give `-d` and `$FLARE_CONTAINER_NAME` as parameters. You need to set the environment variable `FLARE_CONTAINER_NAME` inside the container, again. For instance:

First:

```bash
$ export FLARE_CONTAINER_NAME=flare-process-noaa
```

Then:

```bash
$ /root/flare/flare-container.sh -d $FLARE_CONTAINER_NAME
```


# Run in Debug Mode

To run the scripts in debug mode, just add a `-d` parameter right after the script name. For instance:

```bash
$ wget -O - https://raw.githubusercontent.com/FLARE-forecast/FLARE-containers/$([[ -z "$FLARE_VERSION" ]] && echo 'latest' || echo "$FLARE_VERSION")/commons/flare-install.sh -d | /usr/bin/env bash -s $FLARE_CONTAINER_NAME $([[ -z "$FLARE_VERSION" ]] && echo 'latest' || echo "$FLARE_VERSION")
```
```bash
$ /opt/flare/$FLARE_CONTAINER_NAME/flare-host.sh -d
```