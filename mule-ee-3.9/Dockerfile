###############################################################################
## Dockerizing Mule EE on Raspberry Pi 3
## Version:  1.0
###############################################################################

FROM resin/rpi-raspbian:stretch
MAINTAINER              Simon Valter <simon@valter.info>
# Install base pre-requisites
RUN     apt-get update && apt-get install --no-install-recommends --yes --force-yes apt-utils \
&& apt-get install --no-install-recommends --yes --force-yes unzip \ 
&& apt-get install --no-install-recommends --yes --force-yes curl \
&& apt-get install --no-install-recommends --yes --force-yes jq \
&& echo "deb http://ppa.launchpad.net/webupd8team/java/ubuntu xenial main" | tee /etc/apt/sources.list.d/webupd8team-java.list \
&& echo "deb-src http://ppa.launchpad.net/webupd8team/java/ubuntu xenial main" | tee -a /etc/apt/sources.list.d/webupd8team-java.list \
&& apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys EEA14886 \
&& echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | debconf-set-selections \
&& apt-get update && apt-get install --no-install-recommends --yes --force-yes oracle-java8-installer; exit 0 \
&& rm -rf /var/lib/apt/lists/* && sudo apt-get autoclean
# patch to avoid 404 not found and get latest java
RUN cd /var/lib/dpkg/info && sudo sed -i 's|JAVA_VERSION=8u151|JAVA_VERSION=8u162|' oracle-java8-installer.* \
&& sudo sed -i 's|PARTNER_URL=http://download.oracle.com/otn-pub/java/jdk/8u151-b12/e758a0de34e24606bca991d704f6dcbf/|PARTNER_URL=http://download.oracle.com/otn-pub/java/jdk/8u162-b12/0da788060d494f5095bf8624735fa2f1/|' oracle-java8-installer.* \
&& sudo sed -i 's|SHA256SUM_TGZ="2cca3c6ffcc016ae531802202562ef20630500e134838017a6e91daac2dbb339"|SHA256SUM_TGZ="57720cc98e0dd709e8439df73d70cd1252d76f059e3e08ce0b36e8776b7bfa77"|' oracle-java8-installer.* \
&& sudo sed -i 's|J_DIR=jdk1.8.0_151|J_DIR=jdk1.8.0_162|' oracle-java8-installer.* \
&& apt-get install --no-install-recommends --yes --force-yes oracle-java8-installer \
&& rm -rf /var/lib/apt/lists/* && sudo apt-get autoclean && rm -rf /var/cache/oracle-jdk8-installer

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
ARG     muleVersion=3.9.0
ARG     muleDistribution=mule-ee-distribution-standalone-$muleVersion.zip
ARG     muleHome=/opt/mule-enterprise-standalone-$muleVersion

###############################################################################

## Base container configurations

# Configure instance timezone
RUN     echo 'Europe/Copenhagen' > /etc/timezone && dpkg-reconfigure tzdata


###############################################################################
## MuleEE installation:

## Install Mule EE
WORKDIR /opt/
COPY    ./$muleDistribution /opt/
#RUN     echo "3a49a52f360e5a7e139a654ed698a029 /opt/$muleDistribution" | md5sum -c
RUN     cd /opt && unzip $muleDistribution && ln -s $muleHome/ mule && rm -f $muleDistribution

## [OPTIONAL] Copy the Datamapper plugin
# ADD     ./data-mapper-plugin-$muleVersion.zip /opt/mule/plugins/
ADD     ./wrapper.conf /opt/mule/conf/
ADD     ./mule /opt/mule/bin/
## Copy the License file - pre-package into docker image to avoid leakage
ADD     ./mule-ee-license.lic /opt/mule/conf/
RUN     /opt/mule/bin/mule -installLicense /opt/mule/conf/mule-ee-license.lic && rm -f /opt/mule/conf/mule-ee-license.lic

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
