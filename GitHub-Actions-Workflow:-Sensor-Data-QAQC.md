# GitHub Structure

## Raw Data Branch

xxxx-xxxx-data (e.g. [fcre-catwalk-data](https://github.com/FLARE-forecast/FCRE-data/tree/fcre-catwalk-data))

### Content

**Raw Sensor Data** (e.g. [fcre-waterquality.csv](https://github.com/FLARE-forecast/FCRE-data/blob/fcre-catwalk-data/fcre-waterquality.csv))

_Note:_ The data are directly pushed from the gateways and should not be altered by anyone rather than the gateways.

## QAQC Data Branch

**xxxx-xxxx-data-qaqc** (e.g. [fcre-catwalk-data-qaqc](https://github.com/FLARE-forecast/FCRE-data/tree/fcre-catwalk-data-qaqc))

### Content

**Workflow Files** (e.g. [fcre-catwalk-qaqc.yml](https://github.com/FLARE-forecast/FCRE-data/blob/fcre-catwalk-data-qaqc/.github/workflows/fcre-catwalk-qaqc.yml))

_Note:_ For workflow trigger on push on a specific branch, the workflow file should be in that branch. But, for manual workflow run (i.e. workflow_dispatch), the workflow file should be present in the default branch (i.e. `master` or `main`). If we need both of these triggers, we must have two copies of the same workflow file. To avoid confusion, we recommend not enabling manual workflow run and having just one copy of the workflow file in the raw data branch.

**QAQC R Scripts** (e.g. [R/QAQC_workflow.R](https://github.com/FLARE-forecast/FCRE-data/blob/fcre-catwalk-data-qaqc/R/QAQC_workflow.R))

**Maintenance Logs** (e.g. [CAT_MaintenanceLog.txt](https://github.com/FLARE-forecast/FCRE-data/blob/fcre-catwalk-data-qaqc/CAT_MaintenanceLog.txt))

**QAQC Data** (e.g. [fcre-waterquality-L1.csv](https://github.com/FLARE-forecast/FCRE-data/blob/fcre-catwalk-data-qaqc/fcre-waterquality_L1.csv))