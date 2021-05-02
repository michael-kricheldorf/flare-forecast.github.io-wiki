# Helpful Openwhisk resources

A list of helpful resources people come across for Openwhisk:

* This is a good [introduction to the Openwhisk architecture](https://github.com/apache/openwhisk/blob/master/docs/about.md#how-openWhisk-works)

* I found this to be helpful in understanding how Openwhisk uses its /init and /run API to invoke actions - and all you need is Docker and Python to play with it. [In this document](https://medium.com/openwhisk/advanced-debugging-of-openwhisk-actions-518414636932), essentially invoke.py is performing what an Openwhisk invoker does after a docker container is started

* Understanding and using Docker actions in IBM Bluemix OpenWhisk: https://www.ibm.com/cloud/blog/docker-bluemix-openwhisk
* Docker Action Sample Tutorial: https://jamesthom.as/2017/01/openwhisk-docker-actions/  
* Sample Docker Action: https://stackoverflow.com/questions/39154762/how-do-i-make-a-python-docker-image-an-openwhisk-action  
* `openwhisk/dockerskeleton` Dockerfile: https://hub.docker.com/r/openwhisk/dockerskeleton/dockerfile  
* `python:2.7.12-alpine` Dockerfile: https://github.com/docker-library/python/blob/693a75332e8ae5ad3bfae6e8399c4d7cc3cb6181/2.7/alpine/Dockerfile  

* This is very helpful to build OpenWhisk debug environment: https://github.com/nheidloff/openwhisk-debug-nodejs