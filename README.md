# Docker-Java-Deployment  

### Deploying a simple java dynamic web project to docker image running tomcat server  

###### Building a Dockerfile  

```
FROM ubuntu:16.04
```
Using Ubuntu base image  

```
# Set locales
RUN apt-get clean && apt-get update && apt-get -y install locales && locale-gen en_GB.UTF-8
ENV LANG en_GB.UTF-8
ENV LC_CTYPE en_GB.UTF-8
```
Directly trying to install locales may result in no such dependency errors so add apt-get 
clean before installing

```
ENV DEBIAN_FRONTEND noninteractive
```
Skips any post interactive post installation steps  

```
RUN rm /bin/sh && ln -s /bin/bash /bin/sh
```

In all remotely recent versions of Ubuntu, /bin/sh is just a symbolic link to /bin/dash.  
So make sure that whatever is currently called /bin/sh is backed up somewhere if it's important,  
then delete it and make a new link.  

To remove whatever is currently /bin/sh and restore /bin/sh to what it's supposed to be,   
run the above commands.  

```
# Install dependencies
RUN apt-get update && \
apt-get install -y git build-essential curl wget software-properties-common
```

This will install the essential dependencies  

```
RUN \
echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | debconf-set-selections && \
add-apt-repository -y ppa:webupd8team/java && \
apt-get update && \
apt-get install -y oracle-java8-installer wget unzip tar && \
rm -rf /var/lib/apt/lists/* && \
rm -rf /var/cache/oracle-jdk8-installer

```
This step will accept the licence, updates the repo , installs oracle and removes the repo links  

```
ENV JAVA_HOME /usr/lib/jvm/java-8-oracle
```

Adds the java home path to Environment  

```
# Get Tomcat
RUN wget --quiet --no-cookies http://apache.rediris.es/tomcat/tomcat-8/v${TOMCAT_VERSION}/bin/apache-tomcat-${TOMCAT_VERSION}.tar.gz -O /tmp/tomcat.tgz && \
tar xzvf /tmp/tomcat.tgz -C /opt && \
mv /opt/apache-tomcat-${TOMCAT_VERSION} /opt/tomcat && \
rm /tmp/tomcat.tgz && \ 
rm -rf /opt/tomcat/webapps/examples && \
rm -rf /opt/tomcat/webapps/docs && \
rm -rf /opt/tomcat/webapps/ROOT

```

Downloads the tomcat using wget and add it to a folder /opt/tomcat and removes examples, docs and   
default project from the webapps directory  

```
ADD tomcat-users.xml /opt/tomcat/conf/
```
Adds the tomcat-users.xml to /opt/tomcat/conf/    
Make sure it in the same folder as the Dockerfile    

```
ENV CATALINA_HOME /opt/tomcat
ENV PATH $PATH:$CATALINA_HOME/bin
```

Add the tomcat path to the environment variable and to the system path.  

```
EXPOSE 8080
EXPOSE 8009
```

Exposes ports 8080 and 8009  

```
VOLUME "/opt/tomcat/webapps"
```

Create a persistent volume at /var/lib/docker/volumes

```
WORKDIR /opt/tomcat
```

Sets the current working directory to /opt/tomcat

```
CMD ["/opt/tomcat/bin/catalina.sh", "run"]
```

CMD Runs the commands in the container when it is created and this starts the tomcat server.

### Build an image from the Dockerfile using following command

```
docker build -t ubuntujavatomcat:v1 .
```

Once Completed follow the steps to create a container and add the project to the webapps using docker copy command

```
docker create --name web -p 8080:8080 ubuntujavatomcat:v1
> 7ea043ba2b529a05c5a27ea574ba486a552651d32ee33849bdd135a5259dd622
```

```
docker cp sampleapp.war web:/opt/tomcat/webapps
```

```
docker start web
> web
```

### Now Test the application using curl

```
curl localhost:8080/sampleapp/

> <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
> <html>
> <head>
> <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
> <title>Insert title here</title>
> </head>
> <body>
> <p>Welcome Java to the world of Docker</p> 
> </body>
> </html>

```
