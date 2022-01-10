FROM quay.io/openshift/origin-jenkins-agent-maven:4.6
ADD settings.xml $HOME/.m2/settings.xml
WORKDIR $HOME
COPY pom.xml ./
RUN mvn -T 1C install && rm -rf target
RUN mvn package 
USER 1001
