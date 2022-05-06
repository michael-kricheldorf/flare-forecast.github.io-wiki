# What is RClone?

Rclone is a platform-independent command-line program to manage files on cloud storage. We are interested in its support for S3 object storage, including Minio.

# Install RClone

To install rclone on Linux/macOS/BSD systems, run:

```bash
curl https://rclone.org/install.sh | sudo bash
```

To run rclone on Windows, download it first:

https://rclone.org/downloads/

Then, extract it and find `rclone` executable (`rclone.exe`).

# Configure RClone

Run `rclone config` in the terminal and follow the steps:

```
1- e/n/d/r/c/s/q> n (n for "New remote")
2- name> s3flare (or any other name you prefer)
3- storage> s3
4- provider> Minio
5- env_auth> (Leave it blank and hit enter.)
6- access_key_id> (Enter the access key for FLARE storage on Minio if you have. Leave blank for read-only access.)
7- secret_access_key> (Enter the secret key for FLARE storage on Minio if you have. Leave blank for read-only access.)
8- region> (Leave it blank and hit enter.)
9- endpoint> https://s3.flare.forecast.org
10- location_constraint> (Leave it blank and hit enter.)
11- acl> (Leave it blank and hit enter.)
12- server_side_encryption> (Leave it blank and hit enter.)
13- sse_kms_key_id> (Leave it blank and hit enter.)
14- y/n> (Leave it blank and hit enter.)
15- y/e/d> (Leave it blank and hit enter.)
16- e/n/d/r/c/s/q> q (q for "Quit")
```

# Browse S3 Storage with RClone

You can browse a whole bucket:

```bash
$ rclone lsf s3flare: 
analysis/
drivers/
drivertest/
forecasts/
log/
restart/
scores/
targets/
test/
```

Or browse a specific path:

```bash
$ rclone lsf s3flare:forecasts/sunp
sunp-2021-06-02-sunp_V1.nc
sunp-2021-06-02-sunp_V1.xml
sunp-2021-06-03-sunp_V1.nc
sunp-2021-06-03-sunp_V1.xml
sunp-2021-06-04-sunp_V1.nc
sunp-2021-06-04-sunp_V1.xml
sunp-2021-06-05-sunp_V1.nc
sunp-2021-06-05-sunp_V1.xml
sunp-2021-06-06-sunp_V1.nc
sunp-2021-06-06-sunp_V1.xml
.
.
.
```

# Mount Remote Storage on Linux/macOS/BSD

RClone mount allows mounting any RClone storage as a file system and enables you to work with it the way you work with files and folders on your operating system.

To mount a storage to a path on your operating system, first, you should create an empty local directory on your machine and then mount your remote storage to that directory:

```bash
rclone mount remote:path/to/files /path/to/local/mount
```

For instance:

```bash
mkdir ~/s3flare-local
rclone mount s3flare: ~/s3flare-local &
```

Then, you can work with it as a regular directory with subdirectories and files. You can use your operating system GUI file manager, too.

```bash
$ cd ~/s3flare-local
$ ls
analysis  drivers  drivertest  forecasts  log  restart  scores  targets
```

**Note:** `rclone mount` doesn't stop until you kill it. Adding `&` to the end of the command makes it run in the background.

## Mount in Read-only Mode

If you just need read-only access to the remote storage, mount t in read-only mode to prevent accidental alteration to the files and directories:

```bash
rclone mount --read-only s3flare: ~/s3flare-local &
```