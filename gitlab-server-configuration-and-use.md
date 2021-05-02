This article is about how to configure and use gitlab server, including accounts/repos configured and where to find ssh keys. 

## Configure gitlab

**Pre Step:** Set up the volumes location

```export GITLAB_HOME=/srv/gitlab```

**Step 1:** Install GitLab using Docker Engine

```bash
sudo docker run --detach \
  --hostname gitlab.example.com \
  --publish 443:443 --publish 80:80 --publish 22:22 \
  --name gitlab \
  --restart always \
  --volume $GITLAB_HOME/config:/etc/gitlab \
  --volume $GITLAB_HOME/logs:/var/log/gitlab \
  --volume $GITLAB_HOME/data:/var/opt/gitlab \
  gitlab/gitlab-ee:latest
```

If any of these ports is already in use, we can expose gitlab on different ports. For example, to expose the web interface on the host’s port 8929, and the SSH service on port 2289:

```sh

$ docker run --detach \
   --hostname gitlab.example.com \
   --publish 8929:8929 \
   --publish 2289:22 \
   --name gitlab-test \
   --restart always \
   --volume /srv/gitlab/config:/etc/gitlab \
   --volume /srv/gitlab/logs:/var/log/gitlab \
   --volume /srv/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce:latest

### Modify /etc/gitlab/gitlab.rb in container "gitlab" ###
# log into the container 
docker exec -it gitlab-test /bin/bash
# set the address of gitlab and cancel comment
vi /etc/gitlab/gitlab.rb +32
external_url "http://gitlab.example.com:8929" 

# make nginx to listen on the same port as confgured in external_url
vi /etc/gitlab/gitlab.rb +1300
nginx['listen_port'] = 8929

vi /etc/gitlab/gitlab.rb +593
gitlab_rails['gitlab_shell_ssh_port'] = 2289

### reconfigure gitlab ecosystem ###
$ gitlab-ctl reconfigure
```

**Step 2:** After waiting for several minutes, gitlab server is up. We can visit http://xxx.xxx.xxx.xxx:8929 to access to the gitlab interface. Or you can forward the port to localhost:9999: `ssh -L 9999:localhost:8929 [remote IP address]` .
Here is how we would configure the repositories:

* Every lake name corresponds to one repository on the GitLab server with the same name, and a branch to the container name, and there stores a tar.gz file there that contains just a config.yml file. It is consistent with: https://github.com/FLARE-forecast/FLARE/wiki/Configuration-File. You should modify some parameters like username. 


### Configuration file

```yaml
### Configuration File for CIBR-FLARE Project
## General Settings
git:
  remote:
    server: github.com
    branch: master
    ssh-key-private: /root/.ssh/id_rsa
    user-name: ******
    user-email: ******
## Container Settings
container:
  name: flare-download-noaa
  working-directory:
    pre-run-pull: FALSE
    post-run-push: FALSE
    git:
      remote:
        server: 
        port:
        repository: ****/test-data
        branch: master
        directory: fcre
        ssh-key-private:
        user-name:
        user-email:
  site:
    name: fcre
    latitude: 37.27
    longitude: 79.9
```

In openwhisk mode, we connect to gitlab server to store previous data. We use flare_pullworkdir.sh that does the following: 
* Uses the gitlab server IP, port, lake name, container name, username and SSH key to connect to the GitLab server. We have already sent the SSH key when invoking the openwhisk action. So we can use the key stored in /root/.ssh/id_rsa within the container.
* Checks out the container name branch from the lake name repository
* Unzip/untar the file in a working directory within the container

We also use flare_pushworkdir.sh that does the following:
* Uses the gitlab server IP, port, lake name, container name, username and SSH key to connect to the GitLab server
* Checks out the container name branch from the lake name repository
* tar the downloaded/generated files in the working directory with a daily timestamp, in the format: working_directory_DD_MM_YYYY.tgz. so, every day there will be one working directory file created. 

Source code: https://github.com/Jyuqi/FLARE_DEBUG_NODEJS/tree/master/functions/docker