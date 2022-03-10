# Overview

This document describes how to deploy an OpenWhisk setup on Jetstream2 from scratch. This setup is the simplest possible OpenWhisk "local" deployment - with one node (m3.quad, or larger) hosting the entire cluster. The instructions follow the documentation for deployment of OpenWhisk using ansible in Ubuntu 18.04. Future documentation will consider a multi-node deployment.

The cluster is setup as follows:

* m3.quad instance
* 80GB volume
* a public IP address
* a security group that allows incoming ssh (port 22 for management), icmp (for ping), http and https (ports 80 and 443, for the OpenWhisk event API). 

# Deploy flare_openwhisk instance (using Exosphere GUI)

To avoid repeating documentation, [refer to this page for documentation on the Exosphere GUI. Before you start the steps in the document below to deploy an instance for your cluster front-end, make note of the following:](https://docs.jetstream-cloud.org/ui/exo/exo/)

* When launching an instance, select Ubuntu 18.04 image
* Select the m3.quad flavor (or larger)
* Select Custom disk size, and enter 80GB (or larger) in the text box
* Keep the 1 instance selection unchanged
* Under "Advanced options", click "Show"
* Under "Public IP address", select "Assign a public IP address to this instance"
* Choose your SSH key, or upload a new one

After you click "Create", the instance will be built and deployed within a few minutes




