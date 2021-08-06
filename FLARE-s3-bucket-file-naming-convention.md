# This document describes the file naming convention of NOAAGEFS forecasts that are available on the FLARE-CIBR s3 bucket

## The files are organized using a prefix and a file name, where the prefix follows the structure:

`drivers/noaa_model/NOAAGEFS_Xhr/site/date/cycle/*.nc`

### Where


`noaa_model` = the model of the output, `noaa` for 35-day forecasts or `noaa-point` for 16-day forecasts

`X` = the `noaa_hour`, either 6 or 1, representing the time frequency of the model output

`site` = 4-digit, lowercase site name (e.g. 'bvre')

`date` = start date of the NOAA forecast, YYYY-MM-DD 

`cycle` = the hour of the forecast cycle: 00, 06, 12, 18


## And the individual file name, *.nc, follows the structure
`NOAAGEFS_Xhr_site_dateT00_end_dateTcycle_ens.nc`
### Where
`X` = the `noaa_hour`, either 6 or 1, representing the time frequency of the model output
`end_date` = the end date of the forecast, either 16 or 35 days after the forecast start date


## An example file name
`drivers/noaa/NOAAGEFS_6hr/sunp/2021-07-06/12/NOAAGEFS_6hr_sunp_2021-07-06T00_2021-07-22T12_ens02.nc`

## Important notes
* The '00' ensemble always has an `end_date` 16-days in the future
* The later cycles (06, 12, 18) have an end_date 16-days in the future

## Directory Tree Structure

Here is the directory tree structure for two sample days:

```
.
├── NOAAGEFS_1hr
│   ├── bvre
│   │   ├── 2021-08-04
│   │   │   ├── 00
│   │   │   ├── 06
│   │   │   ├── 12
│   │   │   └── 18
│   │   └── 2021-08-05
│   │       ├── 00
│   │       ├── 06
│   │       ├── 12
│   │       └── 18
│   ├── ccre
│   │   ├── 2021-08-04
│   │   │   ├── 00
│   │   │   ├── 06
│   │   │   ├── 12
│   │   │   └── 18
│   │   └── 2021-08-05
│   │       ├── 00
│   │       ├── 06
│   │       ├── 12
│   │       └── 18
│   ├── fcre
│   │   ├── 2021-08-04
│   │   │   ├── 00
│   │   │   ├── 06
│   │   │   ├── 12
│   │   │   └── 18
│   │   └── 2021-08-05
│   │       ├── 00
│   │       ├── 06
│   │       ├── 12
│   │       └── 18
│   ├── feea
│   │   ├── 2021-08-04
│   │   │   ├── 00
│   │   │   ├── 06
│   │   │   ├── 12
│   │   │   └── 18
│   │   └── 2021-08-05
│   │       ├── 00
│   │       ├── 06
│   │       ├── 12
│   │       └── 18
│   └── sunp
│       ├── 2021-08-04
│       │   ├── 00
│       │   ├── 06
│       │   ├── 12
│       │   └── 18
│       └── 2021-08-05
│           ├── 00
│           ├── 06
│           ├── 12
│           └── 18
├── NOAAGEFS_6hr
│   ├── bvre
│   │   ├── 2021-08-04
│   │   │   ├── 00
│   │   │   ├── 06
│   │   │   ├── 12
│   │   │   └── 18
│   │   └── 2021-08-05
│   │       ├── 00
│   │       ├── 06
│   │       ├── 12
│   │       └── 18
│   ├── ccre
│   │   ├── 2021-08-04
│   │   │   ├── 00
│   │   │   ├── 06
│   │   │   ├── 12
│   │   │   └── 18
│   │   └── 2021-08-05
│   │       ├── 00
│   │       ├── 06
│   │       ├── 12
│   │       └── 18
│   ├── fcre
│   │   ├── 2021-08-04
│   │   │   ├── 00
│   │   │   ├── 06
│   │   │   ├── 12
│   │   │   └── 18
│   │   └── 2021-08-05
│   │       ├── 00
│   │       ├── 06
│   │       ├── 12
│   │       └── 18
│   ├── feea
│   │   ├── 2021-08-04
│   │   │   ├── 00
│   │   │   ├── 06
│   │   │   ├── 12
│   │   │   └── 18
│   │   └── 2021-08-05
│   │       ├── 00
│   │       ├── 06
│   │       ├── 12
│   │       └── 18
│   └── sunp
│       ├── 2021-08-04
│       │   ├── 00
│       │   ├── 06
│       │   ├── 12
│       │   └── 18
│       └── 2021-08-05
│           ├── 00
│           ├── 06
│           ├── 12
│           └── 18
└── NOAAGEFS_raw
    ├── 2021-08-04
    │   ├── 00
    │   ├── 06
    │   ├── 12
    │   └── 18
    └── 2021-08-05
        ├── 00
        ├── 06
        ├── 12
        └── 18
```