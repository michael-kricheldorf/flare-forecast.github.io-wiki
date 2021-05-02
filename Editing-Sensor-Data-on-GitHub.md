If you need to edit the sensor data already uploaded to the GitHub, for instance to add missing data or remove faulty data, you can do that by cloning the branch, editing the files, staging the changes, Commiting them, and finally pushing them back to the GitHub.

**NOTE 1:** You need "write" access to the repository you from which you want to edit the data. If you need that, ask the CIBR admin.

**NOTE 2:** Some of the repositories may be huge. If you don't limit the clone to a single branch and depth equal to 1, it may take a long time and aquire a large space on your hard drive.

For instance, for `fcre-metstation-data`, to get the branch, run the following command:

```bash
git clone --single-branch --depth=1 -b fcre-metstation-data git@github.com:FLARE-forecast/FCRE-data.git fcre-metstation-data
```
The branch will be downloaded to `fcre-metstation-data` directory. After you edited the files, run the following commands:

```bash
git add .
git commit -m "Update Sensor Data"
git push
```

