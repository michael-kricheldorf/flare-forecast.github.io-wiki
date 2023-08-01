## Setting Up the GitHub Actions Self‐hosted Runner VM

First, you need to bring up a VM in Jetstream 2:

* Login to jetstream2's exosphere web interface
* From the top right drop-down, select “create instance”
* Select ubuntu 22.04
* Give the instance a name, e.g. FLARE-runner
* Select m3.quad as size (it's sufficient for now - can be increased if needed)
* Important!! check on “custom disk size”, set it to 100GB (the 20GB default is a bit too small)
* Select “no” for enable web desktop
* Choose your existing ssh key
* Click on “create”
* Wait for VM to be ready
* write down the IP address - you’ll need it for GitHub (see runner_ip below)

## Setting Up the GitHub Self-hosted Runner

Log in to the VM to setup the runner:

* ssh -i your_jetstream2_key ubuntu@runner_ip

0- Create a "New runner" from [Organization > Settings > Actions > Runners](https://github.com/organizations/FLARE-forecast/settings/actions/runners). It will show you the steps to take similar to the followings:

### Download

1- Create a folder
```
mkdir actions-runner && cd actions-runner
```

2- Download the latest runner package
```
curl -o actions-runner-linux-x64-2.307.1.tar.gz -L https://github.com/actions/runner/releases/download/v2.307.1/actions-runner-linux-x64-2.307.1.tar.gz
```

3- Extract the installer
```
tar xzf ./actions-runner-linux-x64-2.307.1.tar.gz
```

### Configure

4- Create the runner and start the configuration experience
```
./config.sh --url https://github.com/FLARE-forecast --token <your_token_here>
```

5- Last step, run it!
```
./run.sh
```

### Using your self-hosted runner

Use this YAML in your workflow file for each job
```YAML
runs-on: self-hosted
```

## References

[About self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners)

[Adding self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/adding-self-hosted-runners)

[Using self-hosted runners in a workflow](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/using-self-hosted-runners-in-a-workflow)