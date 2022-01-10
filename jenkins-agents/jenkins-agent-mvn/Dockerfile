FROM quay.io/openshift/origin-jenkins-agent-maven:4.6
ADD settings.xml $HOME/.m2/settings.xml
#WORKDIR $HOME
#COPY pom.xml ./
#RUN mvn -e org.apache.maven.plugins:maven-dependency-plugin:2.10:tree -Dverbose=true
#RUN mvn -e dependency:go-offline -B
#RUN mvn package 
#USER 1001

ENV HOME=/home/

#RUN chmod 777 /home/usr/app

RUN mkdir -p $HOME

WORKDIR $HOME

# 1. add pom.xml only here

ADD pom.xml $HOME

# 2. start downloading dependencies

RUN mvn -Dmaven.repo.local=./.m2 install assembly:single


USER 1001
