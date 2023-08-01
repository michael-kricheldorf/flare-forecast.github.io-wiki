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

Log in to the VM to setup the GitHub runner:

* ssh -i your_jetstream2_key ubuntu@runner_ip
* Vahid, continue from here

[About self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/about-self-hosted-runners)

[Adding self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/adding-self-hosted-runners)

[Using self-hosted runners in a workflow](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/using-self-hosted-runners-in-a-workflow)