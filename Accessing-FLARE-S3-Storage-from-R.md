Install the library:
```
install.packages("aws.s3")
```

Load the library:
```
library("aws.s3")
```

Setup the URL for Jetstream data store:
```
Sys.setenv("AWS_S3_ENDPOINT" = "tacc.jetstream-cloud.org:8080/")
```

List all the objects in our "flare" bucket:
```
get_bucket(region="", bucket = 'flare')
```
Download a specific file from the server and save it locally (in this example, "localfile.nc"):
save_object(region="", "drivers/noaa-point/NOAAGEFS_1hr/fcre/2021-05-31/00/NOAAGEFS_1hr_fcre_2021-05-31T00_2021-06-16T00_ens05.nc", file="localfile.nc", bucket="flare")