FROM quay.io/openshift/origin-jenkins-agent-maven:4.9
USER root
ADD settings.xml $HOME/.m2/settings.xml
## borrowed the below code from https://github.com/browserless/chrome/blob/master/Dockerfile ##
ARG USE_CHROME_STABLE

# Application parameters and variables
ENV APP_DIR=/usr/src/app
ENV PLAYWRIGHT_BROWSERS_PATH=${APP_DIR}
ENV CONNECTION_TIMEOUT=60000
ENV CHROME_PATH=/usr/bin/google-chrome
ENV HOST=0.0.0.0
ENV IS_DOCKER=true
ENV LANG="C.UTF-8"
ENV USE_CHROME_STABLE=${USE_CHROME_STABLE}


RUN mkdir -p $APP_DIR &&\
    chown -R root:root $APP_DIR

WORKDIR $APP_DIR

# Install Chrome Stable when specified
RUN if [ "$USE_CHROME_STABLE" = "true" ]; then \
    cd /tmp &&\
    wget -q https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb &&\
    dpkg -i google-chrome-stable_current_amd64.deb;\
  fi

# Build and install external binaries + assets
# NOTE: The `PUPPETEER_VERSION` is needed for production versions
# listed in package.json
RUN if [ "$USE_CHROME_STABLE" = "true" ]; then \
    export CHROMEDRIVER_SKIP_DOWNLOAD=false;\
  else \
    export CHROMEDRIVER_SKIP_DOWNLOAD=true;\
  fi  
  
# Run everything after as non-privileged user.

