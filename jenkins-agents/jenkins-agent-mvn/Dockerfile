FROM quay.io/openshift/origin-jenkins-agent-maven:4.6
ADD settings.xml $HOME/.m2/settings.xml
WORKDIR $HOME

# 1. add pom.xml only here

ADD pom.xml $HOME/pom.xml
RUN mvn org.apache.maven.plugins:maven-dependency-plugin:2.10:tree
# 2. start downloading dependencies
RUN mvn dependency:resolve


USER 1001
