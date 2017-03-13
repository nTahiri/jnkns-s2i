FROM centos:centos7

# Jenkins image for OpenShift
#
# This image provides a Jenkins server, primarily intended for integration with
# OpenShift v3.
#
# Volumes:
# * /var/jenkins_home
# Environment:
# * $JENKINS_PASSWORD - Password for the Jenkins 'admin' user.


# Jenkins LTS packages from
# https://pkg.jenkins.io/redhat-stable/
ENV JENKINS_VERSION=2.32.3 \
    HOME=/var/lib/jenkins \
    JENKINS_HOME=/var/lib/jenkins \
    JENKINS_UC=https://updates.jenkins-ci.org

ENV MAVEN_VERSION=3.3.9 \
    JAVA_VERSION=8u45 \
    JAVA_VERSION_PREFIX=1.8.0_45

ENV JAVA_HOME=/opt/jdk$JAVA_VERSION_PREFIX \
    M2_HOME=/home/user/apache-maven-$MAVEN_VERSION
ENV PATH=$JAVA_HOME/bin:$M2_HOME/bin:$PATH
ENV LANG=en_US.UTF-8


LABEL k8s.io.description="Jenkins is a continuous integration server" \
      k8s.io.display-name="Jenkins 2.32.3" \
      openshift.io.expose-services="8080:http" \
      openshift.io.tags="builder,jenkins,jenkins2,ci" \
      io.openshift.s2i.scripts-url=image:///usr/libexec/s2i

# 8080 for main web interface, 50000 for slave agents
EXPOSE 8080 50000

USER root

RUN yum update -y yum.noarch

RUN curl https://pkg.jenkins.io/redhat-stable/jenkins.repo -o /etc/yum.repos.d/jenkins.repo && \
    rpm --import https://pkg.jenkins.io/redhat-stable/jenkins-ci.org.key && \
    yum install -y centos-release-scl-rh && \
    INSTALL_PKGS="rsync gettext git tar zip unzip java-1.8.0-openjdk java-1.8.0-openjdk-devel jenkins-2.32.3 wget" && \
    yum -y --setopt=tsflags=nodocs install $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS && \
    yum clean all  && \
    localedef -f UTF-8 -i en_US en_US.UTF-8 && \
    set -o pipefail && curl -L https://github.com/openshift/origin/releases/download/v1.4.1/openshift-origin-client-tools-v1.4.1-3f9807a-linux-64bit.tar.gz | \
    tar -zx && \
    mv openshift*/oc /usr/local/bin && \
    rm -rf openshift-origin-client-tools-*

RUN mkdir /home/user
RUN mkdir /home/user/apache-maven-$MAVEN_VERSION

RUN wget --no-cookies --no-check-certificate --header "Cookie: oraclelicense=accept-securebackup-cookie" -qO- \
  "http://download.oracle.com/otn-pub/java/jdk/$JAVA_VERSION-b14/jdk-$JAVA_VERSION-linux-x64.tar.gz" | tar -zx -C /opt/ && \
  wget -qO- "http://apache.ip-connect.vn.ua/maven/maven-3/$MAVEN_VERSION/binaries/apache-maven-$MAVEN_VERSION-bin.tar.gz" | tar -zx --strip-components=1 -C /home/user/apache-maven-$MAVEN_     VERSION/


COPY ./contrib/openshift /opt/openshift
COPY ./contrib/jenkins /usr/local/bin
ADD ./contrib/s2i /usr/libexec/s2i


RUN chown -R 1001:0  /opt/openshift && chown -R 1001:0 /var/lib/jenkins && chown -R 1001:0  /var/log && chown -R 1001:0 /usr/libexec/s2i && chown -R 1001:0 /dev

RUN chmod -R 777  /opt/openshift && chmod -R 777 /var/lib/jenkins && chmod -R 777 /var/log && chmod -R 777 /usr/libexec/s2i && \
        chmod 777 /usr/local/bin/install-plugins.sh && chmod 777 /usr/local/bin/fix-permissions && \
        chmod 777 /usr/libexec/s2i/run && chmod 777 /usr/libexec/s2i/assemble && chmod 777 /usr/libexec/s2i/usage

RUN chmod +x /usr/local/bin/install-plugins.sh && chmod +x /usr/local/bin/fix-permissions
RUN chgrp -R 0 /var/log && chmod -R g+rw /var/log && chmod -R g+x /var/log
RUN chgrp -R 0 /opt/openshift && chmod -R g+rw /opt/openshift && chmod -R g+x /opt/openshift


RUN /usr/local/bin/install-plugins.sh /opt/openshift/base-plugins.txt && \
    # need to create <plugin>.pinned files when upgrading "core" plugins like credentials or subversion that are bundled with the jenkins server
    # Currently jenkins v2 does not embed any plugins, but for reference:
    # touch /opt/openshift/plugins/credentials.jpi.pinned && \
    rmdir /var/log/jenkins && \
    chown -R 1001:0 /opt/openshift && \
    /usr/local/bin/fix-permissions /opt/openshift && \
    /usr/local/bin/fix-permissions /var/lib/jenkins


VOLUME ["/var/lib/jenkins"]

USER 1001

CMD ["/usr/libexec/s2i/run"]
