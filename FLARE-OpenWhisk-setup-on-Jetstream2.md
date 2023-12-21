# Overview

This document describes how to deploy an OpenWhisk setup on Jetstream2 from scratch. This setup is the simplest possible OpenWhisk "local" deployment - with one node (m3.quad, or larger) hosting the entire cluster. The instructions follow the documentation for deployment of OpenWhisk using kind and Helm in Ubuntu 22.04. Future documentation will consider a multi-node deployment.

This document is based on the following OpenWhisk documentation: 

* https://github.com/apache/openwhisk-deploy-kube/blob/master/README.md
* https://github.com/apache/openwhisk/blob/master/docs/actions.md
* https://github.com/apache/openwhisk-cli/releases

The cluster is setup as follows:

* m3.quad instance
* 100GB volume
* a public IP address

# Deploy flare_openwhisk instance (using Exosphere GUI)

To avoid repeating documentation, [refer to this page for documentation on the Exosphere GUI. Before you start the steps in the document below to deploy an instance for your cluster front-end, make note of the following:](https://docs.jetstream-cloud.org/ui/exo/exo/)

* When launching an instance, select Ubuntu 22.04 image
* Provide a name (e.g. flare_openwhisk)
* Select the m3.quad flavor (or larger)
* Select Custom disk size, and enter 100GB (or larger) in the text box
* Keep the 1 instance selection unchanged
* Under "Advanced options", click "Show"
* Under "Public IP address", select "Assign a public IP address to this instance"
* Choose your SSH key, or upload a new one

After you click "Create", the instance will be built and deployed within a few minutes

# Connect to flare_openwhisk instance (using SSH)

Once your instance is up and running, it will show "Ready" in the Jetstream2 Exosphere GUI. Click on it to show the public IP address (e.g. 149.165.xxx.yyy - we'll refer to it as PubIP). Use your ssh key to log in and set an initial firewall that allows SSH, HTTP, and HTTPS only:

```
ssh -i your_private_key ubuntu@Pub_IP
sudo ufw allow 22/tcp
sudo ufw allow http
sudo ufw allow https
sudo ufw allow 31001/tcp
sudo ufw enable
```

# Install and run kind (using terminal+ssh)

These commands install kind and deploy a Kubernetes cluster in the node using kind:

```
ssh -i your_private_key ubuntu@Pub_IP
go install sigs.k8s.io/kind@v0.20.0
git clone https://github.com/apache/openwhisk-deploy-kube
export PATH=$PATH:/home/ubuntu/go/bin
cd openwhisk-deploy-kube/
./deploy/kind/start-kind.sh
```

# Install kubectl

These commands install and test kubectl:

```
sudo snap install kubectl --classic
kubectl cluster-info --context kind-kind
kubectl describe nodes
```

# Install helm

```
sudo snap install helm --classic
helm repo add openwhisk https://openwhisk.apache.org/charts
helm repo update
```

# Configure and deploy the OpenWhisk cluster

First, cd into the kind directory and edit the mycluster.yaml file to override cluster defaults:

```
cd ~/openwhisk-deploy-kube/deploy/kind
vi mycluster.yaml
```

The mycluster.yaml file comes with pre-set credentials that are insecure. *You must delete the default auth entries for whisk.system and guest, and add your own auth key! Replace xxxx...:zzz... with your own auth key*

Here you can also change resource limits (e.g. max 2GB memory and 120-minute timeout in this case) 

```
  auth:
    system: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx:zzzz....zzzz"
  limits:
    actions:
      time:
        min: "100ms"
        max: "120m"
        std: "1m"
      memory:
        min: "128m"
        max: "2048m"
        std: "256m"
  containerPool:
    userMemory: "8192m"
```

Now deploy the cluster with helm:

```
helm install owdev openwhisk/openwhisk -n openwhisk --create-namespace -f mycluster.yaml
helm status owdev -n openwhisk
```

# Install wsk

```
cd
wget https://github.com/apache/openwhisk-cli/releases/download/1.2.0/OpenWhisk_CLI-1.2.0-linux-amd64.tgz
tar -xzf OpenWhisk_CLI-1.2.0-linux-amd64.tgz
sudo cp wsk /usr/local/bin/
wsk -i
```

# Delete guest user and create new user

```
kubectl -n openwhisk  -ti exec owdev-wskadmin -- wskadmin user delete guest
kubectl -n openwhisk  -ti exec owdev-wskadmin -- wskadmin user create faasr
```

Take note of the AUTH key that the second wskadmin command shows - you will need it to use the cluster! It will look something like:

```
xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx:yyyyyy...yyy
```

# Configure wsk properties

Replace the xxx...:yyy in the command below with the key for the user newly created from the wskadmin command above:

```
wsk property set --apihost PubIP:31001
wsk property set --auth xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx:yyyyyy...yyy
```

# Register and invoke test action

Edit and save the example file below as as greeting.js:

```
/**
 * @params is a JSON object with optional fields "name" and "place".
 * @return a JSON object containing the message in a field called "msg".
 */
function main(params) {
  // log the parameters to stdout
  console.log('params:', params);

  // if a value for name is provided, use it else use a default
  var name = params.name || 'stranger';

  // if a value for place is provided, use it else use a default
  var place = params.place || 'somewhere';

  // construct the message using the values for name and place
  return {msg:  'Hello, ' + name + ' from ' + place + '!'};
}
```

Register and invoke it as an action:

```
wsk -i action create greeting greeting.js
wsk -i action invoke /faasr/greeting --result
```

# Terminating the cluster

```
helm uninstall owdev -n openwhisk
```

# Still TBD

* Adding X509 certificate to nginx
* Installing alarm package

