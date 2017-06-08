###############################################################################
## Dockerizing Mule EE on Raspberry Pi 3
## Version:  1.0
###############################################################################

FROM resin/rpi-raspbian:jessie
MAINTAINER              Simon Valter <simon@valter.info>

# Install dependencies and setup environment variables
RUN echo "deb http://ppa.launchpad.net/webupd8team/java/ubuntu xenial main" | tee /etc/apt/sources.list.d/webupd8team-java.list
RUN echo "deb-src http://ppa.launchpad.net/webupd8team/java/ubuntu xenial main" | tee -a /etc/apt/sources.list.d/webupd8team-java.list
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys EEA14886
RUN echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | debconf-set-selections
RUN apt-get update && apt-get install -y oracle-java8-installer \
&&  rm -rf /var/lib/apt/lists/*

# Define working directory
WORKDIR /data

# Define commonly used JAVA_HOME variable
ENV JAVA_HOME /usr/lib/jvm/java-8-oracle

# Set locales
RUN locale-gen en_GB.UTF-8
ENV LANG en_GB.UTF-8
ENV LC_CTYPE en_GB.UTF-8

###############################################################################
## Setting up the arguments
ARG     muleVersion=3.8.4
ARG     muleDistribution=mule-ee-distribution-standalone-$muleVersion.zip
ARG     muleHome=/opt/mule-enterprise-standalone-$muleVersion

###############################################################################

## Base container configurations

# Configure instance timezone
RUN     echo 'Europe/Copenhagen' > /etc/timezone && dpkg-reconfigure tzdata

# Install base pre-requisites
RUN     apt-get update
RUN     apt-get upgrade --yes --force-yes
RUN     apt-get install --yes --force-yes apt-utils
RUN     apt-get install --yes --force-yes unzip
RUN     apt-get install --yes --force-yes curl
RUN     apt-get install --yes --force-yes jq

###############################################################################
## MuleEE installation:

## Install Mule EE
WORKDIR /opt/
COPY    ./$muleDistribution /opt/
#RUN     echo "3a49a52f360e5a7e139a654ed698a029 /opt/$muleDistribution" | md5sum -c
RUN     cd /opt && unzip $muleDistribution
RUN     ls
RUN     ln -s $muleHome/ mule
RUN     ls -l mule
RUN     rm -f $muleDistribution

## [OPTIONAL] Copy the Datamapper plugin
# ADD     ./data-mapper-plugin-$muleVersion.zip /opt/mule/plugins/
ADD     ./wrapper.conf /opt/mule/conf/
ADD     ./mule /opt/mule/bin/
## Copy the License file - pre-package into docker image to avoid leakage
# ADD     ./mule-ee-license.lic /opt/mule/conf/
# RUN     /opt/mule/bin/mule -installLicense /opt/mule/conf/mule-ee-license.lic
# RUN     rm -f /opt/mule/conf/mule-ee-license.lic

## Copy the mule start/stop script
ADD     ./startMule.sh /opt/mule/bin/
RUN     chmod 755 /opt/mule/bin/startMule.sh

###############################################################################
## Configure mule runtime access pre-requisites

## HTTPS Port for Anypoint Platform communication
EXPOSE  443

## Mule remote debugger
EXPOSE  5000

## Mule JMX port (must match Mule config file)
EXPOSE  1098

## Mule Cluster ports
EXPOSE 5701
EXPOSE 54327

###############################################################################
## Expose the necessary port ranges as required by the apps to be deployed

## HTTP Service Port
EXPOSE 8081

## HTTPS Service Port
EXPOSE 8091

###############################################################################

## Environment and execution:
ENV             MULE_BASE /opt/mule
WORKDIR         /opt/mule/bin
ENTRYPOINT      ["/opt/mule/bin/startMule.sh"]
