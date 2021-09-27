# What is RClone?

Rclone is a platform-independent command-line program to manage files on cloud storage. It also has a GUI. We are interested in its support for S3 object stores including Minio.

# Install RClone

To install rclone on Linux/macOS/BSD systems, run:

```bash
curl https://rclone.org/install.sh | sudo bash
```

To run rclone on Windows, download it first:

https://rclone.org/downloads/

Then, extract it and find `rclone` executable.

# Configure RClone

Run `rclone config` in the terminal and follow the steps:

```
1- e/n/d/r/c/s/q> n (n for "New remote")
2- name> s3jetstream (or any other name you prefer)
3- storage> s3
4- provider> Minio
5- env_auth> (Leave it blank and hit enter.)
6- access_key_id> (Enter the access key you should already have for Jetstream storage on Minio.)
7- secret_access_key> (Enter the secret key you should already have for Jetstream storage on Minio.)
8- region> (Leave it blank and hit enter.)
9- endpoint> https://tacc.jetstream-cloud.org:8080
10- location_constraint> (Leave it blank and hit enter.)
11- acl> (Leave it blank and hit enter.)
12- server_side_encryption> (Leave it blank and hit enter.)
13- sse_kms_key_id> (Leave it blank and hit enter.)
14- y/n> (Leave it blank and hit enter.)
15- e/n/d/r/c/s/q> q (q for "Quit")
```

# Run RClone WebUI

To run RClone WebUI and explore the files from your web browser, run:

```bash
rclone rcd --rc-web-gui --rc-no-auth
```

A browser tab should open and you can navigate to "Explorer", choose the new remote you just created and explore the files.

![RClone WebUI Explorer](https://raw.githubusercontent.com/rclone/rclone-webui-react/master/screenshots/remoteexplorer.png)

**Note:** It shouldn't ask for a username and password. But due to a bug, it may do. In that case, stop the WebUI and close all the tabs in the browser and run the WebUI again.