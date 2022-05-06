FROM quay.io/openshift/origin-jenkins-agent-maven:4.9

ENV HOME = $HOME/.m2

WORKDIR $HOME

ADD settings.xml $HOME/settings.xml

ADD pom.xml pom.xml

RUN mvn dependency:resolve

USER 1001
