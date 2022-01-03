## Non-root User

```
PATH=/home/ubuntu/bin:/home/ubuntu/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
00,20,40 * * * * timeout 1500 docker run --rm -v ~/flare-host/shared:/home/user/flare/shared flareforecast/flare-process-noaa:21.01.5 /home/user/flare/flare-container.sh flare-process-noaa-fcre
07,27,47 * * * * timeout 1500 docker run --rm -v ~/flare-host/shared:/home/user/flare/shared flareforecast/flare-process-noaa:21.01.5 /home/user/flare/flare-container.sh flare-process-noaa-sunp
14,34,54 * * * * timeout 1500 docker run --rm -v ~/flare-host/shared:/home/user/flare/shared flareforecast/flare-process-noaa:21.01.5 /home/user/flare/flare-container.sh flare-process-noaa-feea
#00 */6 * * * echo $(date) >> ~/flare-host2/flare-process-noaa-grid.log && docker run --rm -v ~/flare-host2/shared/flare-process-noaa-grid-fixed:/root/NOAA flareforecast/flare-process-noaa-grid:21.01.14 /bin/bash -c "cd /root/neon4cast-noaa-download/ && Rscript launch_download_downscale.R" 2>&1 | tee -a ~/flare-host2/flare-process-noaa-grid.log
00 */6 * * * mc cp ~/flare-host/shared/flare-process-noaa-fcre/NOAAGEFS_1hr s3jetstream/flare/drivers/noaa-point/ --recursive --newer-than 4d && mc cp ~/flare-host/shared/flare-process-noaa-fcre/NOAAGEFS_6hr s3jetstream/flare/drivers/noaa-point/ --recursive --newer-than 4d && mc cp ~/flare-host/shared/flare-process-noaa-sunp/NOAAGEFS_1hr s3jetstream/flare/drivers/noaa-point/ --recursive --newer-than 4d && mc cp ~/flare-host/shared/flare-process-noaa-sunp/NOAAGEFS_6hr s3jetstream/flare/drivers/noaa-point/ --recursive --newer-than 4d && mc cp ~/flare-host/shared/flare-process-noaa-feea/NOAAGEFS_1hr s3jetstream/flare/drivers/noaa-point/ --recursive --newer-than 4d && mc cp ~/flare-host/shared/flare-process-noaa-feea/NOAAGEFS_6hr s3jetstream/flare/drivers/noaa-point/ --recursive --newer-than 4d
#00 */6 * * * mc cp ~/flare-host2/shared/flare-process-noaa-grid-fixed/ s3jetstream/flare/drivers/noaa/ --recursive --newer-than 4d && echo "done" >> ~/s3.log
00 * * * * ~/github-monitor/github-monitor1.sh
00 * * * * ~/github-monitor/github-monitor2.sh
00 * * * * ~/github-monitor/github-monitor3.sh
00 * * * * ~/github-monitor/github-monitor4.sh
00 * * * * ~/github-monitor/github-monitor5.sh
00 * * * * ~/github-monitor/github-monitor6.sh
00 * * * * ~/github-monitor/github-monitor7.sh
00 * * * * ~/github-monitor/github-monitor8.sh
00 * * * * ~/github-monitor/github-monitor9.sh
00 * * * * ~/github-monitor/github-monitor10.sh
00 * * * * ~/github-monitor/github-monitor11.sh
00 * * * * ~/github-monitor/github-monitor12.sh
30 08 * * * ~/flare-host/flare-download-observations/flare-host.sh && curl -fsS -m 10 --retry 5 -o /dev/null https://hc-ping.com/b9873223-baa4-422a-b81a-1667b8d44287
35 08 * * * ~/flare-host/flare-process-observations/flare-host.sh && curl -fsS -m 10 --retry 5 -o /dev/null https://hc-ping.com/d049b0d1-f85f-4a3c-97bd-c2f5fe20ad30
45 08 * * * ~/flare-host/flare-generate-forecast/flare-host.sh && curl -fsS -m 10 --retry 5 -o /dev/null https://hc-ping.com/d2bcc8a5-70b9-4ffd-9979-83c222a3a3ef
00 09 * * * ~/flare-host/flare-visualize/flare-host.sh && cp -p "`ls -dtr1 /home/ubuntu/flare-host/shared/flare-visualize/*.png | tail -1`" /home/ubuntu/FLARE-containers/send-email/current-forecast.png && curl -fsS -m 10 --retry 5 -o /dev/null https://hc-ping.com/3236764b-289d-4e91-beb9-956583a2db00

```

## root User

```
-t auto /dev/sdc1 /home/ubuntu/flare-host2
*/5 * * * * /usr/bin/python3 /opt/evio/evio-control.py
00 08,20 * * * cp -r /home/ubuntu/flare-host/shared/flare-process-noaa-fcre/NOAAGEFS_6hr/fcre/2022-* /home/ubuntu/flare-host/shared/flare-process-noaa/NOAAGEFS_6hr/fcre/.
00 08,20 * * * cp -r /home/ubuntu/flare-host/shared/flare-process-noaa-fcre/NOAAGEFS_1hr/fcre/2022-* /home/ubuntu/flare-host/shared/flare-process-noaa/NOAAGEFS_1hr/fcre/.
30 11 * * * cp -p "`ls -dtr1 /home/ubuntu/flare-host/shared/flare-visualize/*.png | tail -1`" /home/ubuntu/FLARE-containers/send-email/current-forecast.png && docker run --rm -v /home/ubuntu/FLARE-containers/send-email:/home/user/s-nail/ flareforecast/send-email:latest ./send_email.sh FCR
31 11 * * * docker run --rm -v /home/ubuntu/FLARE-containers/send-email:/home/user/s-nail/ flareforecast/send-email:latest ./send_email.sh SUNP
```