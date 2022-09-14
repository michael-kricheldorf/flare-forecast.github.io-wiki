# Access Resources

## JS2 VM

### Cron Jobs

#### Root User

```
@reboot sleep 30 && mount /dev/sdb /home/ubuntu/applications/github-backup/head-backup/
@reboot sleep 30 && mount /dev/sdc /home/ubuntu/applications/github-backup/full-backup/
```

#### Non-root User

```
@reboot sleep 30 && /usr/bin/rclone mount --read-only s3flare: ~/s3flare-directory
00,14 11 * * * ~/applications/forecast/forecast-fcre.sh
07,21 11 * * * ~/applications/forecast/forecast-sunp.sh 
28 11 * * * ~/applications/daily-email/daily-email-fcre.sh
30 11 * * * ~/applications/daily-email/daily-email-sunp.sh
00 * * * * ~/applications/github-monitor/github-monitor1.sh
00 * * * * ~/applications/github-monitor/github-monitor2.sh
00 * * * * ~/applications/github-monitor/github-monitor3.sh
00 * * * * ~/applications/github-monitor/github-monitor4.sh
00 * * * * ~/applications/github-monitor/github-monitor5.sh
00 * * * * ~/applications/github-monitor/github-monitor6.sh
00 * * * * ~/applications/github-monitor/github-monitor7.sh
00 * * * * ~/applications/github-monitor/github-monitor8.sh
00 * * * * ~/applications/github-monitor/github-monitor9.sh
00 * * * * ~/applications/github-monitor/github-monitor10.sh
00 * * * * ~/applications/github-monitor/github-monitor11.sh
00 * * * * ~/applications/github-monitor/github-monitor12.sh
00 06 15 * * ~/applications/github-backup/head-backup/github-head-backup.sh
00 06 01 * * ~/applications/github-backup/full-backup/github-full-backup.sh

```

## Sensor Gateways

### Evio Access from JS2 VM

```bash
ssh carina
ssh miaa
ssh bita
ssh bjorn
ssh annie
```

### Reverse SSH Access from JS2 VM:

```bash
ssh carina-ssh
ssh miaa-ssh
ssh bita-ssh
ssh bjorn-ssh
ssh annie-ssh
```

# Common Issues

## Daily Emails

### Water temperature forecast graph is not attached to FCRE or SUNP daily emails

**Reason:** Forecast has not been run in the expected time on JS2 VM, usually due to missing NOAA forecasts or disk space problems.

**Potential Solution/Workaround:** Re-ran the forecast and watch for the error logs.

#### How to troubleshoot

**On JS2 VM:**

Check availability of NOAA forecasts:

```bash
ll ~/s3flare-directory/drivers/noaa/NOAAGEFS_6hr/fcre/
ll ~/s3flare-directory/drivers/noaa/NOAAGEFS_6hr/sunp/
```

If required to skip days in FCRE forecast (usually due to missing NOAA forecast), `start_datetime` and `forecast_start_datetime` need to be changed in the following config file:

```
~/s3flare-directory/restart/fcre/fcre_js2/configure_run.yml
```

The same applies to SUNP forecast and the following config file:

```
~/s3flare-directory/restart/sunp/sunp_V3/configure_run.yml
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