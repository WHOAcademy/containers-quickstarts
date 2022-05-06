FROM quay.io/openshift/origin-jenkins-agent-base:4.9

USER root

RUN curl http://www.apache.org/dist//maven/binaries/apache-maven-3.0.3-bin.tar.gz && \
    tar -zxvf apache-maven-3.0.3-bin.tar.gz

RUN echo "export M2_HOME=/opt/apache-maven-3.0.3" >> .bashrc && \
    echo "export PATH=${M2_HOME}/bin:${PATH}" >> .bashrc

RUN mvn -version

USER 1001
