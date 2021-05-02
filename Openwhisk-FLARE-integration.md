# Evolving guidelines for how to integrate FLARE with Openwhisk

## Container image

It seems important that our FLARE container images are able to run in two modes of operation:

### Manual mode (no Openwhisk integration)

Here, a user would run the container from the command-line of a host. This would be helpful for development, debugging, and to run FLARE (or individual FLARE modules) not in an "online forecast" mode. 

In this mode, I think we can safely assume that there is only one FLARE instance/lake in use in the host, that all containers run on the same host (e.g. an user's desktop), and that host volumes can be mounted in the container. The event/action here is essentially equivalent to a user manually running the container. Configuration can be passed to the container by docker run.

(This is essentially the mode Vahid has been developing containers as of May 2020)

### Openwhisk mode

Here, containers are started/destroyed by the Openwhisk invoker, and actions run inside of them. The container is strictly stateless - there is no option to mount volumes - and an Openwhisk cluster should be able to handle multiple FLARE instances for multiple lakes.

In this mode, the container image needs to have a Web service endpoint (e.g. FLASK) compliant with the Openwhisk REST API (/init, /run) to invoke actions. Since there is no mounted volume, the configuration is passed through the Openwhisk API (dictionary).

While in manual mode we can use a mounted volume for performance enhancements (e.g. local files that were downloaded from git repositories pulled in previous invocations), in Openwhisk mode we'll need to figure out a different mechanism. We can use gitlab containers running locally in the same LAN as the Openwhisk cluster to store this state. 

### Volume mounts vs. save/restore

In manual mode, the assumption of running in a single host with a mounted volume facilitates the deployment by an end-user - there is no need to configure a storage service. In Openwhisk mode, we can use the storage service to: 1) pack the contents of a working directory that might be used for another container downstream, 2) push it to a local storage service, 3) pull from a local storage service, and 4) unpack. Steps 1) and 2) happen at the end of the container's execution, while steps 3) and 4) happen at startup. In this way, we can emulate the behavior of the host volume mount, and avoid changes to the rest of the container.

For instance, when running in manual mode, a docker container starts with:

```
docker run -v ${DIRECTORY_HOST_SHARED}/${CONTAINER}:${DIRECTORY_CONTAINER_SHARED} ${DOCKERHUB_ID}/${CONTAINER} ${DIRECTORY_CONTAINER}/${CONTAINER_SCRIPT}
```

In manual mode, the ${DIRECTORY_CONTAINER_SHARED} directory is setup as a result of docker run invocation, whereas in Openwhisk mode, - v cannot be used. Instead, the /run action within the container can proceed as follows:

* pull a tar.gz ball from a LAN repository
* untar it in ${DIRECTORY_CONTAINER_SHARED}
* (run flare-container.sh)
* create a tar.gz from ${DIRECTORY_CONTAINER_SHARED}
* push tar.gz ball to LAN repository

We need to be mindful of where to store and how to name these files that carry over from one container. One possibility could be:

* use per-lake repository names, e.g. fcre-working-files
* use per-forecast branch names, such as a [standard date/time format](https://en.wikipedia.org/wiki/ISO_8601), e.g. 2020-05-22T00:00Z
* use per-container file names, e.g. flare-external-driver-interface-noaa.tgz

With this naming approach, a single Openwhisk cluster could run multiple lakes concurrently. We could have not only daily forecasts, but support other periods as well. We can have a cleanup container that removes old working files from storage

### Dealing with ssh keys

One of the key parameters we pass from host to container is ssh keys to authenticate to git server(s). In the absence of a host mount, it seems like one possibility would be to store these keys in CouchDB, as long as an Openwhisk action can retrieve them from the database and pass along to the /run API endpoint

### Container image design

It should be possible to have a single image that can be used to run in both modes:

* In manual mode, the REST API is basically unused, and the container runs the FLARE code upon container startup, configured from docker run CLI and gets configuration and mounted volumes also from CLI. 

* In Openwhisk mode, the container does not run FLARE code upon container startup - FLARE code is invoked through the API stub upon receiving the /init REST call from Openwhisk. This may involve some pre-FLARE-execution steps (e.g. download a snapshot of git local files), and post-FLARE-execution steps (e.g. upload new snapshot of git local files)


## Near-term tasks:

### Test manual invocation of a .js action without Openwhisk

#### Task 1: manual invoke, simple Javascript action

Follow the instructions [in this tutorial](https://medium.com/openwhisk/advanced-debugging-of-openwhisk-actions-518414636932) in your own computer. You should be able to start the openwhisk/nodejs6action Dockerhub container, and get to the point where you can init and run an action:

```
> invoke.py init hello.js 
{"OK":true}

> invoke.py run '{"name":"manual invoke"}'
{"greeting":"Hello manual invoke!"}
```

#### Task 2: manual invoke, simple Python action

Repeat task 1, but now use a Python action instead of .js, using the openwhisk/python3action Dockerhub container

#### Task 3: Openwhisk invoke, simple Python action

Repeat task 2, but now invoke using Openwhisk on the ACIS cluster, with something along the lines of:

```
wsk action create <ACTION_NAME> --docker <IMAGE>
```

#### Task 4: Openwhisk invoke, FLARE container, first step

Start with the flareforecast/flare-external-driver-interface-noaa Dockerhub container. 

Then, add Python, FLASK and the actionproxy.py script to get the Openwhisk API endpoints for /init and /run up and running on port 8080. It may be helpful to look at the [Openwhisk "skeleton" container](https://hub.docker.com/r/openwhisk/dockerskeleton) to see how they set up a container that can run an arbitrary executable (e.g. a shell script).

Take a look at actionproxy.py within the "skeleton" container - this is what they use to run code within the container, and we should be able to leverage it as a starting point.

* For the /init web service endpoint: I don't think we need to do anything, since we already have all the code we need inside our
* For the /run endpoint: implement it such that it just runs a simple hello_world.sh shell script (just a simple script that echoes hello world will suffice)

Create your own DockerHub account - you'll need to save and publish this modified container to DockerHub to test invoking from OpenWhisk

Use a wsk action to start your modified container, with no arguments associated with the action. Check that it invokes the hello_world.sh script inside your modified container

#### Task 5: Openwhisk invoke, FLARE container, second step

Extend what you did in task 4, but now instead of running the simple hello_world.sh script, try:

* Passing the following arguments through the /run endpoint: a lake name (a string, e.g. "fcre"), a container name (also a string, e.g. flare-external-driver-interface-noaa), a server IP, a user name, and an SSH key
* Create and run a new script, echo_whisk_arguments.sh script, that reads these arguments, and echoes them out

Publish this new version of the container, and test it again, now invoking with arguments

#### Task 6: Openwhisk invoke, FLARE container, third step

Extend what you did in task 5, but now instead of just echoing arguments out, use them to fetch data from a Git repository in your GitLab container:

* Create a repository on the GitLab server with the lake name, and a branch with the container name, and store a tar.gz file there that contains just a config.yml file
* Create and run a new script, flare_pullworkdir.sh that does the following: 
* Uses the IP, user name, and SSH key to connect to the GitLab server
* Checks out the container name branch from the lake name repository
* Unzip/untar the file in a working directory within the container
* Echoes the content of the config.yml file

Test this modified container to ensure you can see the config.yml file output

#### Task 7: Openwhisk invoke, FLARE container, fourth step

Extend what you did in task 5, but now instead of just echoing the config.yml file, you should set it up so that it is fed to flare_container.sh. Get a sample config.yml file from Vahid for testing. You should ensure that flare_container.sh runs properly, and that it downloads data from NOAA

#### Task 8: Openwhisk invoke, FLARE container, fifth step

Now develop the flare_pushworkdir.sh script that:

* tar+gzips the working directory (where the NOAA data should have been downloaded to)
* uploads to the repo/branch for this container name

