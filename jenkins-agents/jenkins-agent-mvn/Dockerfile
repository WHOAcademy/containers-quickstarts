FROM quay.io/openshift/origin-jenkins-agent-maven:4.9
ADD settings.xml $HOME/.m2/settings.xml

USER root

# add google-chrome based on https://access.redhat.com/discussions/6114861
COPY google-chrome.repo /tmp/
COPY centos.repo /tmp/

COPY *.repo /etc/yum.repos.d/
COPY RPM-GPG-KEY-CentOS-Official /etc/pki/rpm-gpg/

RUN INSTALL_PKGS="xdg-utils liberation-fonts google-chrome" && \
    rm -f /etc/yum.repos.d/*.repo && \
    mv /tmp/centos.repo /etc/yum.repos.d/centos.repo && \
    mv /tmp/google-chrome.repo /etc/yum.repos.d/google-chrome.repo && \
    dnf -y install $INSTALL_PKGS && \
    dnf -y clean all

USER 1001