FROM ubuntu:latest

RUN apt-get install -y software-properties-common
RUN apt-get update -y
RUN apt-get install -y openssl openssh-client tar xz-utils wget git curl zip bzip2 
RUN rm -rf /var/lib/apt/lists/*

###########################################################################################################
# APT-FAST
###########################################################################################################

RUN add-apt-repository ppa:saiarcot895/myppa
RUN apt-get update -y
RUN apt-get -y install apt-fast

###########################################################################################################
# JAVA 8
###########################################################################################################

RUN echo -e "" | add-apt-repository ppa:webupd8team/java
RUN apt-get update -y 
RUN echo debconf shared/accepted-oracle-license-v1-1 select true | sudo debconf-set-selections
RUN echo debconf shared/accepted-oracle-license-v1-1 seen true | sudo debconf-set-selections
RUN apt-get install -y oracle-java8-installer
RUN sudo apt-get install -y oracle-java8-set-default

###########################################################################################################
# JENKINS
###########################################################################################################

ENV JENKINS_HOME /home/jenkins
ENV JENKINS_SLAVE_AGENT_PORT 50000

# Jenkins is run with user `jenkins`, uid = 1000
# If you bind mount a volume from the host or a data container, 
# ensure you use the same uid
RUN mkdir -p /home/jenkins
RUN useradd -c "jenkins" jenkins -u 1000 -s /bin/bash -d "/home/jenkins"
RUN /bin/echo -e "jenkins\njenkins" | passwd jenkins

# Jenkins home directory is a volume, so configuration and build history 
# can be persisted and survive image upgrades

RUN echo "JAVA_HOME=/usr/lib/jvm/java-8-oracle/bin" >> /home/jenkins/.bsharc
RUN echo "export JAVA_HOME" >> /home/jenkins/.bsharc
RUN echo 'PATH=$PATH:$JAVA_HOME' >> /home/jenkins/.bsharc
RUN echo "export PATH" >> /home/jenkins/.bsharc
RUN chown jenkins -Rf /home/jenkins/

# `/usr/share/jenkins/ref/` contains all reference configuration we want 
# to set on a fresh new installation. Use it to bundle additional plugins 
# or config file with your custom jenkins Docker image.
RUN mkdir -p /usr/share/jenkins/ref/init.groovy.d

ENV TINI_SHA 066ad710107dc7ee05d3aa6e4974f01dc98f3888



ENV JENKINS_VERSION 1.642.1
ENV JENKINS_SHA 6a0213256670a00610a3e09203850a0fcf1a688e


# could use ADD but this one does not check Last-Modified header 
# see https://github.com/docker/docker/issues/8331
RUN curl -fL http://mirrors.jenkins-ci.org/war-stable/$JENKINS_VERSION/jenkins.war -o /usr/share/jenkins/jenkins.war \
  && echo "$JENKINS_SHA /usr/share/jenkins/jenkins.war" | sha1sum -c -

ENV JENKINS_UC https://updates.jenkins-ci.org
RUN chown -Rf jenkins "$JENKINS_HOME" /usr/share/jenkins/ref

# for main web interface:
EXPOSE 8080

# will be used by attached slave agents:
EXPOSE 50000

ENV COPY_REFERENCE_FILE_LOG $JENKINS_HOME/copy_reference_file.log

# from a derived Dockerfile, can use `RUN plugins.sh active.txt` to setup /usr/share/jenkins/ref/plugins from a support bundle
COPY plugins.sh /usr/local/bin/plugins.sh
COPY plugins.txt /usr/share/jenkins/plugins.txt
RUN /usr/local/bin/plugins.sh /usr/share/jenkins/plugins.txt

COPY jenkins.sh /usr/local/bin/jenkins.sh
RUN chmod +x /usr/local/bin/jenkins.sh
RUN chown -Rf jenkins /home/jenkins/
RUN chmod -Rf 777 /home/jenkins/

###########################################################################################################
# GENERATE A SSH KEY
###########################################################################################################
RUN mkdir -p /home/jenkins/.ssh/ && \
    cd /home/jenkins/.ssh/ && \
    ssh-keygen -t rsa -b 4096 -C '' -f /home/jenkins/.ssh/id_rsa && \
    ssh-agent /bin/bash && eval `ssh-agent -s` && \
    ssh-add /home/jenkins/.ssh/id_rsa && \
    ls -lha /home/jenkins/ && \
    cat /home/jenkins/.ssh/id_rsa.pub && \
    echo "StrictHostKeyChecking no" >>  /home/jenkins/.ssh/config && \
    echo "UserKnownHostsFile /dev/null" >>  /home/jenkins/.ssh/config

###########################################################################################################
# NODEJS
###########################################################################################################

# gpg keys listed at https://github.com/nodejs/node
RUN set -ex \
  && for key in \
    9554F04D7259F04124DE6B476D5A82AC7E37093B \
    94AE36675C464D64BAFA68DD7434390BDBE9B9C5 \
    0034A06D9D9B0064CE8ADF6BF1747F4AD2306D93 \
    FD3A5288F042B6850C66B31F09FE44734EB7990E \
    71DCFD284A79C3B38668286BC97EC7A07EDE3FC1 \
    DD8F2338BAE7501E3DD5AC78C273792F7D83545D \
    B9AE9905FFD7803F25714661B63B535A4C206CA9 \
    C4F0DFFF4E8C1A8236409D08E73BC641CC11F4C8 \
  ; do \
    gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$key"; \
  done

ENV NPM_CONFIG_LOGLEVEL info
ENV NODE_VERSION 5.6.0

RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz"
RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc"
RUN gpg --verify SHASUMS256.txt.asc
RUN grep " node-v$NODE_VERSION-linux-x64.tar.xz\$" SHASUMS256.txt.asc | sha256sum -c -
RUN tar -xJf "node-v$NODE_VERSION-linux-x64.tar.xz" -C /usr/local --strip-components=1
RUN rm "node-v$NODE_VERSION-linux-x64.tar.xz" SHASUMS256.txt.asc

RUN ln -s /usr/bin/nodejs /usr/bin/node
RUN chown -Rf jenkins /usr/local/lib/node_modules


###########################################################################################################
# PHP 7
###########################################################################################################

RUN apt-get install -y language-pack-en-base
RUN LC_ALL=en_US.UTF-8 add-apt-repository ppa:ondrej/php-7.0

RUN apt-get -y update && apt-get -y upgrade && apt-get -y install snmp php7.0 \
    php7.0-bz2  \
    php7.0-cli  \
    php7.0-common  \
    php7.0-curl  \
    php7.0-dbg  \
    php7.0-dev  \
    php7.0-gd  \
    php7.0-imap  \
    php7.0-interbase  \
    php7.0-intl  \
    php7.0-json  \
    php7.0-ldap  \
    php7.0-mcrypt  \
    php7.0-mysql  \
    php7.0-odbc  \
    php7.0-opcache  \
    php7.0-pgsql  \
    php7.0-phpdbg  \
    php7.0-pspell  \
    php7.0-readline  \
    php7.0-recode  \
    php7.0-snmp  \
    php7.0-sqlite3  \
    php7.0-sybase  \
    php7.0-tidy  \
    php7.0-xmlrpc \
    php-mongodb \
    php-redis \
    php-apcu \
    php-amqp \
    php-memcached

RUN sudo pecl install mongodb
RUN echo "extension=mongodb.so" > /etc/php/7.0/cli/conf.d/20-mongodb.ini

###########################################################################################################
# SQL LITE
###########################################################################################################
RUN apt-get install sqlite3 libsqlite3-dev

###########################################################################################################
# ANSIBLE
###########################################################################################################
RUN apt-add-repository -y ppa:ansible/ansible
RUN apt-get -y update
RUN apt-get -y install ansible openssh-client

###########################################################################################################
# ANSISTRANO
###########################################################################################################
RUN ansible-galaxy install --force carlosbuenosvinos.ansistrano-deploy carlosbuenosvinos.ansistrano-rollback
RUN adduser --disabled-password --gecos '' ansistrano
RUN adduser ansistrano sudo
RUN echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers



###########################################################################################################
# CHANGE TO JENKINS USER
###########################################################################################################

USER jenkins
ENTRYPOINT ["/usr/local/bin/jenkins.sh"]
 
