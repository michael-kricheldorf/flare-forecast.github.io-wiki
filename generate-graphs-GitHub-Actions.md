For each lake, a GitHub Action is responsible to run the `generate-graphs` R script and generate the output graphs. Then, a container downloads the generated graphs and attaches them to the daily email.

## wvwa-generate-graphs (Western Virginia Water Authority - FCRE, BVRE, and CCRE Sensor Check)

### Script

The `generate-graphs` R script can be edited by anyone with write access:

https://github.com/FLARE-forecast/FCRE-data/blob/wvwa-generate-graphs/wvwa-generate-graphs.R

### Output Graphs

The graphs are generated upon running the R script:

https://github.com/FLARE-forecast/FCRE-data/tree/wvwa-graphs

### Workflow File

The workflow file contains the actions and settings for running the GitHub Action. It currently runs the R script every day at 7:00 and saves the output graphs.

https://github.com/FLARE-forecast/FCRE-data/blob/master/.github/workflows/wvwa-generate-graphs.yml

## sunp-generate-graphs (SUNP Sensor Check) 

### Script

The `generate-graphs` R script can be edited by anyone with write access:

https://github.com/FLARE-forecast/SUNP-data/blob/sunp-generate-graphs/sunp-generate-graphs.R

### Output Graphs

The graphs are generated upon running the R script:

https://github.com/FLARE-forecast/SUNP-data/tree/sunp-graphs

### Workflow File

The workflow file contains the actions and settings for running the GitHub Action. It currently runs the R script every day at 7:00 and saves the output graphs.

https://github.com/FLARE-forecast/SUNP-data/blob/master/.github/workflows/sunp-generate-graphs.yml