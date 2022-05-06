FROM quay.io/openshift/origin-jenkins-agent-maven:4.9

ADD settings.xml $HOME/.m2/settings.xml

ADD pom.xml $HOME/.m2/pom.xml

RUN mvn dependency:resolve
