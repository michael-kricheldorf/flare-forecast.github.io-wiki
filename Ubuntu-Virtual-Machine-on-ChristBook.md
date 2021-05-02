# SCC_IPOP VM on ChristBook

**IP Address:** 192.168.10.2

## Carina

**Connect to Carina**

`ssh scc@192.168.10.11`

**Collect System Monitoring Data**

`/home/scc/applications/monitor.sh`

**Push Data to GitHub**

`/home/scc/applications/git.sh`

## Mia

**Connect to Mia**

`ssh scc@192.168.10.12`

**Collect System Monitoring Data**

`/home/scc/applications/monitor.sh`

**Push Data to GitHub**

`/home/scc/applications/git.sh`

**Pull Data from George**

`sudo python3 /home/scc/applications/RM-Gateway.py`

## Bruno

**Connect to Bruno**

`ssh scc@192.168.10.1`

**Get NOAA Data and Push Them to GitHub**

`/home/scc/applications/noaa-git.sh`

**Check Sensors**

`Rscript /home/scc/applications/daily-email/SensorCheckScript.R`

**Prepare Data for Daily Email**

`/home/scc/applications/daily-email/daily-email-git.sh`

**Check On-demand Push Requests**

`/home/scc/applications/on-demand-push/check-push-requests.sh`

**Prepare Git Log for Daily Email**

`sudo /home/scc/applications/git-log.sh`

**Send Daily Email**

`sudo /home/scc/applications/daily-email/daily-email-send.sh`