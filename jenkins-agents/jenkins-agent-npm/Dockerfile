# Builder
FROM registry.access.redhat.com/ubi9/ubi:9.8 AS builder

ARG YQ_VERSION=v4.50.1
ARG JQ_VERSION=1.6

RUN curl -L \
    "https://github.com/mikefarah/yq/releases/download/${YQ_VERSION}/yq_linux_amd64" \
    -o /usr/local/bin/yq \
    && chmod +x /usr/local/bin/yq \
    && yq --version

RUN curl -L \
    "https://github.com/stedolan/jq/releases/download/jq-${JQ_VERSION}/jq-linux64" \
    -o /usr/local/bin/jq \
    && chmod +x /usr/local/bin/jq \
    && jq --version


# Jenkins agent
FROM ghcr.io/redhat-cop/containers-quickstarts/jenkins-agent:v4.18

ARG NODEJS_VERSION=24.22.1

LABEL name="redhat-cop/jenkins-agent-npm" \
      io.k8s.display-name="Jenkins Agent npm" \
      io.k8s.description="The jenkins agent npm." \
      com.redhat.component="redhat-cop/containers-quickstarts/jenkins-agent-npm"

ENV NPM_CONFIG_PREFIX="$HOME/.npm-global" \
    PATH="$HOME/node_modules/.bin/:$HOME/.npm-global/bin/:$PATH"

USER root

COPY package.json /tmp/package.json

RUN NODEJS_TARBALL="node-v${NODEJS_VERSION}-linux-x64.tar.gz" \
    && cd /tmp \
    && curl -fsSLO "https://nodejs.org/dist/v${NODEJS_VERSION}/${NODEJS_TARBALL}" \
    && curl -fsSL "https://nodejs.org/dist/v${NODEJS_VERSION}/SHASUMS256.txt" \
       | grep " ${NODEJS_TARBALL}\$" | sha256sum -c - \
    && tar -xzf "${NODEJS_TARBALL}" -C /usr/local --strip-components=1 --no-same-owner \
    && rm -f "${NODEJS_TARBALL}" \
    && npm install -g /tmp

RUN chown -R 1001:0 /home/jenkins/.npm \
    && chmod -R 775 /home/jenkins/.npm

COPY --from=builder /usr/local/bin/yq /usr/local/bin/yq
COPY --from=builder /usr/local/bin/jq /usr/local/bin/jq

USER 1001

RUN yq --version \
    && jq --version \
    && npm --version \
    && node --version
