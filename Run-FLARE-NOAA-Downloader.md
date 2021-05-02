**Note:** This is a temporary method to manually run FLARE NOAA downloader script on the host until we create a new container based on this setup.

1- Get the NOAA downloader script:

```bash
$ wget https://raw.githubusercontent.com/FLARE-forecast/FLARE-containers/master/flare-download-noaa/QueuedDownloader.py
```

2- Run it for today and the last 3 days one by one.

Here is the usage help:

```bash
Usage: QueuedDownloader.py base_directory date lat lon

Example: QueuedDownloader.py /opt/flare/shared/flare-download-noaa 20210224 255 160
```

For instance, for Feb 24, 2021 and store the data in `/opt/flare/share/flare-download-noaa`:

```bash
$ python3 QueuedDownloader.py /opt/flare/shared/flare-download-noaa 20210224 255 160
```

You also need to run it for the last 3 days:

```bash
$ python3 QueuedDownloader.py /opt/flare/shared/flare-download-noaa 20210223 255 160
$ python3 QueuedDownloader.py /opt/flare/shared/flare-download-noaa 20210222 255 160
$ python3 QueuedDownloader.py /opt/flare/shared/flare-download-noaa 20210221 255 160
```

Each run usually takes about 30 minutes.

You should see the outputs files in the output directory:

```bash
$ ls /opt/flare/shared/flare-download-noaa
20210221  20210222  20210223  20210224

```
And for each day:

```bash
$ ls /opt/flare/shared/flare-download-noaa/20210224
'gefs_pgrb2ap5_all_00z.ascii?apcpsfc[0:30][0:64][255][160]'   'gefs_pgrb2ap5_all_06z.ascii?pressfc[0:30][0:64][255][160]'   'gefs_pgrb2ap5_all_12z.ascii?ugrd10m[0:30][0:64][255][160]'
'gefs_pgrb2ap5_all_00z.ascii?dlwrfsfc[0:30][0:64][255][160]'  'gefs_pgrb2ap5_all_06z.ascii?rh2m[0:30][0:64][255][160]'      'gefs_pgrb2ap5_all_12z.ascii?vgrd10m[0:30][0:64][255][160]'
'gefs_pgrb2ap5_all_00z.ascii?dswrfsfc[0:30][0:64][255][160]'  'gefs_pgrb2ap5_all_06z.ascii?tmp2m[0:30][0:64][255][160]'     'gefs_pgrb2ap5_all_18z.ascii?apcpsfc[0:30][0:64][255][160]'
'gefs_pgrb2ap5_all_00z.ascii?pressfc[0:30][0:64][255][160]'   'gefs_pgrb2ap5_all_06z.ascii?ugrd10m[0:30][0:64][255][160]'   'gefs_pgrb2ap5_all_18z.ascii?dlwrfsfc[0:30][0:64][255][160]'
'gefs_pgrb2ap5_all_00z.ascii?rh2m[0:30][0:64][255][160]'      'gefs_pgrb2ap5_all_06z.ascii?vgrd10m[0:30][0:64][255][160]'   'gefs_pgrb2ap5_all_18z.ascii?dswrfsfc[0:30][0:64][255][160]'
'gefs_pgrb2ap5_all_00z.ascii?tmp2m[0:30][0:64][255][160]'     'gefs_pgrb2ap5_all_12z.ascii?apcpsfc[0:30][0:64][255][160]'   'gefs_pgrb2ap5_all_18z.ascii?pressfc[0:30][0:64][255][160]'
'gefs_pgrb2ap5_all_00z.ascii?ugrd10m[0:30][0:64][255][160]'   'gefs_pgrb2ap5_all_12z.ascii?dlwrfsfc[0:30][0:64][255][160]'  'gefs_pgrb2ap5_all_18z.ascii?rh2m[0:30][0:64][255][160]'
'gefs_pgrb2ap5_all_00z.ascii?vgrd10m[0:30][0:64][255][160]'   'gefs_pgrb2ap5_all_12z.ascii?dswrfsfc[0:30][0:64][255][160]'  'gefs_pgrb2ap5_all_18z.ascii?tmp2m[0:30][0:64][255][160]'
'gefs_pgrb2ap5_all_06z.ascii?apcpsfc[0:30][0:64][255][160]'   'gefs_pgrb2ap5_all_12z.ascii?pressfc[0:30][0:64][255][160]'   'gefs_pgrb2ap5_all_18z.ascii?ugrd10m[0:30][0:64][255][160]'
'gefs_pgrb2ap5_all_06z.ascii?dlwrfsfc[0:30][0:64][255][160]'  'gefs_pgrb2ap5_all_12z.ascii?rh2m[0:30][0:64][255][160]'      'gefs_pgrb2ap5_all_18z.ascii?vgrd10m[0:30][0:64][255][160]'
'gefs_pgrb2ap5_all_06z.ascii?dswrfsfc[0:30][0:64][255][160]'  'gefs_pgrb2ap5_all_12z.ascii?tmp2m[0:30][0:64][255][160]'
```

Now you can use the download path, `/opt/flare/shared/flare-download-noaa` here, as the value for `read_from_file` in `flare-config.yml` of `flare-process-noaa` container:

```yaml
read_from_path: /root/flare/shared/flare-download-noaa # Path to the downloaded NOAA files. To directly download from NOMADS server, leave the path blank.
```