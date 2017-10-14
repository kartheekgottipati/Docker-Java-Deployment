# Docker-Java-Deployment  
### Deploying a simple java dynamic web project to docker image running tomcat server  
###### Building a Dockerfile

[//]: <> (Using Ubuntu base image)
FROM ubuntu:16.04

[//]: <> (Skips any post interactive post installation steps)
ENV DEBIAN_FRONTEND noninteractive

[//]: <> (In all remotely recent versions of Ubuntu, /bin/sh is just a symbolic link to /bin/dash.)
[//]: <> (So make sure that whatever is currently called /bin/sh is backed up somewhere if it's important,)
[//]: <> (then delete it and make a new link.)
RUN rm /bin/sh && ln -s /bin/bash /bin/sh

  
