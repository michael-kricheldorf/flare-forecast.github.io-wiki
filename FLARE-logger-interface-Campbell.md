## Documentation for FLARE-logger-interface-Campbell container

### Overview of container functionality
This container is responsible for retrieving data from a Campbell Scientific data logger. The container is designed to work with data loggers using the FTP protocol with the following interfaces: Ethernet (wired or Wi-Fi) and USB (Ethernet over USB). Support for RS-232 serial port is not provided. [Information on FTP streaming](https://s.campbellsci.com/documents/us/technical-papers/ftp-streaming.pdf). [Information on Ethernet over USB](https://help.campbellsci.com/CR300/Content/shared/Connect/Config_Ethernet_USB.htm?tocpath=_____7)

### Data logger setup
* Configuring the Campbell data logger for Ethernet connectivity
* Installing software on Campbell data logger
* Configuring data tables 

### Key software dependences

### Trigger
This container is triggered by a timer

### Inputs and outputs
The inputs to this container, provided through the configuration file, are:
* Name to be used for the CSV output file
* Git server endpoint, repository, branch, and credentials to push updates

The container generates a CSV output with a time series of sensor data readings as an output. New readings are appended to this time series, and updates are pushed to a git repository.

### Error logs

### Configuration file
```yaml
flare-logger-interface-campbell:
  datalogger:
    model                 : CR300            # device's model
    interface             : WiredEthernet    # WiredEthernet, WiFi, or USB
    name                  : metsensor        # unique name for the logger; its output will be stored in a branch with the same name in the target git repo
  git:  
    local:                                   # local git repo is mounted from this volume from host
      mounted_volume      : /home/user/flare-logger-interface-campbell                                    
    remote:                                  # server, repo, branch, user where the data is committed to
      server              : github.com
      repository          : mylake/data.git
      user                : gituser
      private-ssh-key     : /home/user/.ssh/id_rsa
```
