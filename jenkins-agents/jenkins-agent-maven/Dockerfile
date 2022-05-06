FROM quay.io/openshift/origin-jenkins-agent-base:4.9

USER root

RUN curl https://downloads.apache.org/maven/binaries/apache-maven-3.2.2-bin.tar.gz --output apache-maven-3.2.2-bin.tar.gz && \
    tar -zxvf apache-maven-3.2.2-bin.tar.gz

RUN echo "export M2_HOME=/opt/apache-maven-3.2.2" >> .bashrc && \
    echo "export PATH=${M2_HOME}/bin:${PATH}" >> .bashrc

RUN mvn -version

USER 1001
