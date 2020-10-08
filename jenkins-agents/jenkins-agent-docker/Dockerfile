FROM docker.io/docker

FROM quay.io/openshift/origin-jenkins-agent-base:4.4


# ADD docker-ce.repo /etc/yum.repos.d/docker-ce.repo
# ADD ubi.repo /etc/yum.repos.d/ubi.repo

# # RUN yum -y install docker

# # RUN yum  -y remove  docker-common docker container-selinux docker-selinux docker-engine

# RUN subscription-manager repos --enable=rhel-7-server-extras-rpms

# RUN yum --disablerepo='rhel-*' -y install docker-ce





# ADD ubi.repo /etc/yum.repos.d/ubi.repo
# USER root
# RUN yum --disablerepo='rhel-*' install -y yum-utils
# yum -y --disablerepo='rhel-*' install --setopt=tsflags=nodocs rh-git218
# RUN yum-config-manager \
#     --add-repo \
#     https://download.docker.com/linux/centos/docker-ce.repo

# RUN yum --disablerepo='rhel-*' install https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm

# RUN yum --disablerepo='rhel-*' install docker-ce docker-ce-cli

# RUN yum remove docker docker-common docker-selinux docker-engine

# RUN yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# RUN yum install docker-ce-stable

# RUN INSTALL_PKGS=" \
#       docker" && \
#     DISABLE_REPOS=--disablerepo='rhel-*' && \
#     yum $DISABLE_REPOS install -y yum-utils && \
#     yum -y --setopt=tsflags=nodocs $DISABLE_REPOS install $INSTALL_PKGS && \
#     rpm -V $INSTALL_PKGS && \
#     yum -y clean all --enablerepo='*'

# ADD scl_enable /usr/share/container-scripts/
# ENV BASH_ENV=/usr/share/container-scripts/scl_enable \
#     ENV=/usr/share/container-scripts/scl_enable \
#     PROMPT_COMMAND=". /usr/share/container-scripts/scl_enable"

USER 1001
