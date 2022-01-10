FROM quay.io/openshift/origin-jenkins-agent-maven:4.6
ADD settings.xml $HOME/.m2/settings.xml
WORKDIR $HOME
COPY pom.xml ./
RUN mvn dependency:resolve
USER 1001
