## Installation

### Windows 10

1. Download MinIO Client (mc) executable:  
https://dl.min.io/client/mc/release/windows-amd64/mc.exe

2. Create MinIO directory at `C:\MinIO`.

3. Move `mc.exe` to `C:\MinIO\mc.exe`.

4. Add `C:\MinIO` to your path:  
https://medium.com/@kevinmarkvi/how-to-add-executables-to-your-path-in-windows-5ffa4ce61a53

### macOS

Install mc packages using Homebrew:

```
brew install minio/stable/mc
```

## Installation Verification

Run `mc` command from terminal (Command Prompt, Terminal, etc.)

```
mc
```

The following output should appear on the screen:

```
NAME:
  mc - MinIO Client for cloud storage and filesystems.

USAGE:
  mc [FLAGS] COMMAND [COMMAND FLAGS | -h] [ARGUMENTS...]

COMMANDS:
  alias      set, remove and list aliases in configuration file
  ls         list buckets and objects
  mb         make a bucket
  rb         remove a bucket
  cp         copy objects
  mirror     synchronize object(s) to a remote site
  cat        display object contents
  head       display first 'n' lines of an object
  pipe       stream STDIN to an object
  share      generate URL for temporary access to an object
  find       search for objects
  sql        run sql queries on objects
  stat       show object metadata
  mv         move objects
  tree       list buckets and objects in a tree format
  du         summarize disk usage recursively
  retention  set retention for object(s)
  legalhold  manage legal hold for object(s)
  diff       list differences in object name, size, and date between two buckets
  rm         remove objects
  version    manage bucket versioning
  ilm        manage bucket lifecycle
  encrypt    manage bucket encryption config
  event      manage object notifications
  watch      listen for object notification events
  undo       undo PUT/DELETE operations
  anonymous  manage anonymous access to buckets and objects
  tag        manage tags for bucket and object(s)
  replicate  configure server side bucket replication
  admin      manage MinIO servers
  update     update mc to latest release

GLOBAL FLAGS:
  --autocompletion              install auto-completion for your shell
  --config-dir value, -C value  path to configuration folder (default: "C:\\Users\\vdane\\mc")
  --quiet, -q                   disable progress bar display
  --no-color                    disable color theme
  --json                        enable JSON lines formatted output
  --debug                       enable debug output
  --insecure                    disable SSL certificate verification
  --help, -h                    show help
  --version, -v                 print the version

TIP:
  Use 'mc --autocompletion' to enable shell autocompletion

VERSION:
  RELEASE.2021-07-27T06-46-19Z
```

## Configuration

The following configuration is required to be done once.

Add Jetstream S3 Alias:

```
mc alias set s3jetstream https://tacc.jetstream-cloud.org:8080 --api S3v2 --path auto
```

It has no Access Key and Secret Key. So just hit Enter for Access Key and Secret Key:

```
←[33;3mEnter Access Key: ←[0m
←[33;3mEnter Secret Key: ←[0m
Added `s3jetstream` successfully.
```

Verify the `s3jetstream` alias has been added successfully:

```
mc alias list
```

The output should be as following:
```
gcs
  URL       : https://storage.googleapis.com
  AccessKey : YOUR-ACCESS-KEY-HERE
  SecretKey : YOUR-SECRET-KEY-HERE
  API       : S3v2
  Path      : dns

local
  URL       : http://localhost:9000
  AccessKey :
  SecretKey :
  API       :
  Path      : auto

play
  URL       : https://play.min.io
  AccessKey : XXXXX
  SecretKey : xxxxx
  API       : S3v4
  Path      : auto

s3
  URL       : https://s3.amazonaws.com
  AccessKey : YOUR-ACCESS-KEY-HERE
  SecretKey : YOUR-SECRET-KEY-HERE
  API       : S3v4
  Path      : dns

s3jetstream
  URL       : https://tacc.jetstream-cloud.org:8080
  AccessKey :
  SecretKey :
  API       :
  Path      : auto
```

## Using MinIO Cient

Browse the bucket:

```
mc ls s3jetstream\flare
```

Output:
```
[2021-08-09 13:32:09 EDT]     0B drivers\
```

Command:
```
mc ls s3jetstream\flare\drivers
```

Output:
```
[2021-08-09 13:34:09 EDT]     0B noaa-point\
[2021-08-09 13:34:09 EDT]     0B noaa\
```

Command:
```
\Users\vdane>mc ls s3jetstream\flare\drivers\noaa-point
```

Output:
```
[2021-08-09 13:35:38 EDT]     0B NOAAGEFS_1hr\
[2021-08-09 13:35:38 EDT]     0B NOAAGEFS_6hr\
```

Command:
```
mc ls s3jetstream\flare\drivers\noaa\
```

Output:
```
[2021-08-09 13:36:42 EDT]     0B NOAAGEFS_1hr\
[2021-08-09 13:36:42 EDT]     0B NOAAGEFS_6hr\
[2021-08-09 13:36:42 EDT]     0B NOAAGEFS_raw\
```

Command:
```
mc ls s3jetstream\flare\drivers\noaa\NOAAGEFS_1hr
```

Output:
```
[2021-08-09 13:37:57 EDT]     0B bvre\
[2021-08-09 13:37:57 EDT]     0B ccre\
[2021-08-09 13:37:57 EDT]     0B fcre\
[2021-08-09 13:37:57 EDT]     0B feea\
[2021-08-09 13:37:57 EDT]     0B sunp\
```

Command:
```
mc ls s3jetstream\flare\drivers\noaa\NOAAGEFS_1hr\fcre
```

Output
```
[2021-08-09 13:39:08 EDT]     0B 2021-05-26\
[2021-08-09 13:39:08 EDT]     0B 2021-05-27\
[2021-08-09 13:39:08 EDT]     0B 2021-05-28\
[2021-08-09 13:39:08 EDT]     0B 2021-05-29\
[2021-08-09 13:39:08 EDT]     0B 2021-05-30\
[2021-08-09 13:39:08 EDT]     0B 2021-05-31\
[2021-08-09 13:39:08 EDT]     0B 2021-06-01\
[2021-08-09 13:39:08 EDT]     0B 2021-06-02\
[2021-08-09 13:39:08 EDT]     0B 2021-06-03\
[2021-08-09 13:39:08 EDT]     0B 2021-06-04\
[2021-08-09 13:39:08 EDT]     0B 2021-06-05\
[2021-08-09 13:39:08 EDT]     0B 2021-06-06\
[2021-08-09 13:39:08 EDT]     0B 2021-06-07\
[2021-08-09 13:39:08 EDT]     0B 2021-06-08\
[2021-08-09 13:39:08 EDT]     0B 2021-06-09\
[2021-08-09 13:39:08 EDT]     0B 2021-06-10\
[2021-08-09 13:39:08 EDT]     0B 2021-06-11\
[2021-08-09 13:39:08 EDT]     0B 2021-06-12\
[2021-08-09 13:39:08 EDT]     0B 2021-06-13\
[2021-08-09 13:39:08 EDT]     0B 2021-06-14\
[2021-08-09 13:39:08 EDT]     0B 2021-06-15\
[2021-08-09 13:39:08 EDT]     0B 2021-06-16\
[2021-08-09 13:39:08 EDT]     0B 2021-06-17\
[2021-08-09 13:39:08 EDT]     0B 2021-06-18\
[2021-08-09 13:39:08 EDT]     0B 2021-06-19\
[2021-08-09 13:39:08 EDT]     0B 2021-06-20\
[2021-08-09 13:39:08 EDT]     0B 2021-06-21\
[2021-08-09 13:39:08 EDT]     0B 2021-06-22\
[2021-08-09 13:39:08 EDT]     0B 2021-06-23\
[2021-08-09 13:39:08 EDT]     0B 2021-06-24\
[2021-08-09 13:39:08 EDT]     0B 2021-06-25\
[2021-08-09 13:39:08 EDT]     0B 2021-06-26\
[2021-08-09 13:39:08 EDT]     0B 2021-06-27\
[2021-08-09 13:39:08 EDT]     0B 2021-06-28\
[2021-08-09 13:39:08 EDT]     0B 2021-06-29\
[2021-08-09 13:39:08 EDT]     0B 2021-06-30\
[2021-08-09 13:39:08 EDT]     0B 2021-07-01\
[2021-08-09 13:39:08 EDT]     0B 2021-07-02\
[2021-08-09 13:39:08 EDT]     0B 2021-07-03\
[2021-08-09 13:39:08 EDT]     0B 2021-07-04\
[2021-08-09 13:39:08 EDT]     0B 2021-07-05\
[2021-08-09 13:39:08 EDT]     0B 2021-07-06\
[2021-08-09 13:39:08 EDT]     0B 2021-07-07\
[2021-08-09 13:39:08 EDT]     0B 2021-07-08\
[2021-08-09 13:39:08 EDT]     0B 2021-07-25\
[2021-08-09 13:39:08 EDT]     0B 2021-07-26\
[2021-08-09 13:39:08 EDT]     0B 2021-07-27\
[2021-08-09 13:39:08 EDT]     0B 2021-07-29\
[2021-08-09 13:39:08 EDT]     0B 2021-08-02\
[2021-08-09 13:39:08 EDT]     0B 2021-08-05\
```