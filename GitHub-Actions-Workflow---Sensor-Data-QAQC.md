# GitHub Structure

## Raw Data Branch
xxxx-xxxx-data (e.g. [fcre-catwalk-data](https://github.com/FLARE-forecast/FCRE-data/tree/fcre-catwalk-data))

### Content

Raw sensor data directly pushed from the gateways (e.g. [fcre-waterquality.csv](https://github.com/FLARE-forecast/FCRE-data/blob/fcre-catwalk-data/fcre-waterquality.csv))

**Note:** Should not be altered by anyone rather than the gateways.

## QAQC Data Branch

xxxx-xxxx-data-qaqc (e.g. [fcre-catwalk-data-qaqc](https://github.com/FLARE-forecast/FCRE-data/tree/fcre-catwalk-data-qaqc))

### Content

Workflow files (e.g. [fcre-catwalk-qaqc.yml](https://github.com/FLARE-forecast/FCRE-data/blob/fcre-catwalk-data-qaqc/.github/workflows/fcre-catwalk-qaqc.yml))

QAQC R scripts (e.g. [R/QAQC_workflow.R](https://github.com/FLARE-forecast/FCRE-data/blob/fcre-catwalk-data-qaqc/R/QAQC_workflow.R))

Maintenance logs (e.g. [CAT_MaintenanceLog.txt](https://github.com/FLARE-forecast/FCRE-data/blob/fcre-catwalk-data-qaqc/CAT_MaintenanceLog.txt))

QAQC Data (e.g. [fcre-waterquality-L1.csv](https://github.com/FLARE-forecast/FCRE-data/blob/fcre-catwalk-data-qaqc/fcre-waterquality_L1.csv))