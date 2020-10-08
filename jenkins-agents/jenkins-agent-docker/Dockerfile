FROM quay.io/openshift/origin-jenkins-agent-base:4.4

USER root

COPY ubi.repo docker-ce.repo /etc/yum.repos.d/

RUN DISABLE_REPOS=--disablerepo='rhel-*' && \
    yum $DISABLE_REPOS --setopt=tsflags=nodocs install -y http://mirror.centos.org/centos/7/extras/x86_64/Packages/container-selinux-2.107-1.el7_6.noarch.rpm && \
    yum clean all -y && \
    rm -rf /var/cache/yum

RUN DISABLE_REPOS=--disablerepo='rhel-*' && \
    yum $DISABLE_REPOS --setopt=tsflags=nodocs install -y http://mirror.centos.org/centos/7/os/x86_64/Packages/libseccomp-2.3.1-4.el7.x86_64.rpm && \
    yum clean all -y && \
    rm -rf /var/cache/yum


RUN DISABLE_REPOS=--disablerepo='rhel-*' && \
    yum $DISABLE_REPOS --setopt=tsflags=nodocs install -y docker-ce  && \
    yum clean all -y && \
    rm -rf /var/cache/yum

CMD systemctl start docker

USER 1001
