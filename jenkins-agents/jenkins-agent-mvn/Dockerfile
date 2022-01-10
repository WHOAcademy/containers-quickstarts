FROM quay.io/openshift/origin-jenkins-agent-maven:4.6
ADD settings.xml $HOME/.m2/settings.xml
#WORKDIR $HOME
#COPY pom.xml ./
#RUN mvn -e org.apache.maven.plugins:maven-dependency-plugin:2.10:tree -Dverbose=true
#RUN mvn -e dependency:go-offline -B
#RUN mvn package 
#USER 1001

ENV HOME=/home/usr/app

RUN mkdir -p $HOME

WORKDIR $HOME

# 1. add pom.xml only here

ADD pom.xml $HOME

# 2. start downloading dependencies

RUN ["/usr/local/bin/mvn-entrypoint.sh", "mvn", "verify", "clean", "--fail-never"]

# 3. add all source code and start compiling

ADD . $HOME

RUN ["mvn", "package"]

EXPOSE 8005

CMD ["java", "-jar", "./target/dist.jar"]

USER 1001
