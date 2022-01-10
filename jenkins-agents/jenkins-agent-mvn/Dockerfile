FROM quay.io/openshift/origin-jenkins-agent-maven:4.6
ADD settings.xml $HOME/.m2/settings.xml
WORKDIR $HOME
COPY C:/Users/Pallavi.Das/Documents/GitHub/WHOA-LXP-TestAutomationSuite/WHOA_Suite/pom.xml .
RUN mvn dependency:go-offline
USER 1001
