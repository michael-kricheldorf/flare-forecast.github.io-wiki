# Common Issues

## Daily Emails

### Water temperature forecast graph is not attached to FCRE or SUNP daily emails

**Reason:** Forecast has not been run in the expected time on JS2 VM, usually due to missing NOAA forecasts or disk space problems.

**Potential Solution/Workaround:** Re-ran the forecast and watch for the error logs.

#### How to troubleshoot

**On JS2 VM:**

Check availability of NOAA forecasts:

```bash
ll s3flare-directory/drivers/noaa/NOAAGEFS_6hr/fcre/
ll s3flare-directory/drivers/noaa/NOAAGEFS_6hr/sunp/
```

Check disk space:

```bash
df -h /dev/sda1
```

Manually run forecasts:

```bash
~/applications/forecast/forecast-fcre.sh
~/applications/forecast/forecast-sunp.sh
```

### Sensor Check Graphs are not attached to FCRE or SUNP daily emails

**Reason:** GitHub Action has not been run in the expected time on GitHub, usually due to GitHub server problems.

**Potential Solution/Workaround:** Re-ran the GitHub Action and watch for the error logs.

## Healthchecks.io Monitors

### All the monitors are down at the same time.

**Reason:** Monitor scripts have not been successfully executed in the expected time on JS2, usually due to JS2 network problems.

**Potential Solution/Workaround:** Check network connection on JS2 and wait for it to get resolved.

**On JS2 VM:**

```bash
ping hc-ping.com
```