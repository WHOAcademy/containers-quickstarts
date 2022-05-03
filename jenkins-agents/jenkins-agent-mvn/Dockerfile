FROM quay.io/openshift/origin-jenkins-agent-maven:4.9.0
ENV HOME = $HOME/.m2
WORKDIR $HOME
ADD settings.xml $HOME/settings.xml


# 1. add pom.xml only here
ADD pom.xml pom.xml

# 2. start downloading dependencies
RUN mvn dependency:resolve

USER 1001
