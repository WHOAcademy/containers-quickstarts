FROM quay.io/openshift/origin-jenkins-agent-maven:4.6
ADD settings.xml $HOME/.m2/settings.xml
WORKDIR $HOME
COPY pom.xml ./
RUN mvn -e org.apache.maven.plugins:maven-dependency-plugin:2.10:tree -Dverbose=true
RUN mvn -e dependency:go-offline -B
RUN mvn package 
USER 1001
