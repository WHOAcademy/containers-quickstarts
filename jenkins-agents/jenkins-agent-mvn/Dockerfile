FROM quay.io/openshift/origin-jenkins-agent-maven:4.9
USER root
ADD settings.xml $HOME/.m2/settings.xml
## borrowed the below code from https://github.com/browserless/chrome/blob/master/Dockerfile ##
#ARG USE_CHROME_STABLE
ARG CHROME_VERSION=92.0.4515.107
# Application parameters and variables
ENV APP_DIR=/usr/src/app
ENV PLAYWRIGHT_BROWSERS_PATH=${APP_DIR}
ENV CONNECTION_TIMEOUT=60000
ENV CHROME_PATH=/usr/bin/google-chrome
ENV HOST=0.0.0.0
ENV IS_DOCKER=true
ENV LANG="C.UTF-8"
ENV CHROME_VERSION=${CHROME_VERSION}


RUN mkdir -p $APP_DIR &&\
    chown -R root:root $APP_DIR

WORKDIR $APP_DIR
RUN dnf update -y && \
    dnf install maven unzip wget -y

# Get and install LATEST_RELEASE of ChromeDriver
RUN wget -O /tmp/chromedriver.zip https://chromedriver.storage.googleapis.com/${CHROME_VERSION}/chromedriver_linux64.zip && \
    unzip /tmp/chromedriver.zip && \
    mv chromedriver /usr/bin

# Install Google Chrome (same version as ChromeDriver)
RUN wget https://dl.google.com/linux/linux_signing_key.pub && \
    rpm --import linux_signing_key.pub && \
    dnf install --skip-broken https://dl.google.com/linux/chrome/rpm/stable/x86_64/google-chrome-stable-${CHROME_VERSION}-1.x86_64.rpm -y


# Run everything after as non-privileged user.

USER 1001

