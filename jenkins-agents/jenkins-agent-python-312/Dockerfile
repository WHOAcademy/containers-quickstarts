# NOTE: ghcr.io/redhat-cop/containers-quickstarts/jenkins-agent:v4.18 is built on
# RHEL 8.10 (glibc 2.28, OpenSSL 1.1.1k). Every stage uses that same image so the
# CPython we compile links against exactly the libraries present at runtime.
# A UBI9 builder would produce binaries that need glibc 2.34 / OpenSSL 3 and fail.

# Tools builder: static yq/jq binaries, checksum-verified
FROM ghcr.io/redhat-cop/containers-quickstarts/jenkins-agent:v4.18 AS tools

ARG YQ_VERSION=v4.50.1
ARG YQ_SHA256=c7a1278e6bbc4924f41b56db838086c39d13ee25dcb22089e7fbf16ac901f0d4
ARG JQ_VERSION=1.7.1
ARG JQ_SHA256=5942c9b0934e510ee61eb3e30273f1b3fe2590df93933a93d7c58b81d19c8ff5

USER root

RUN curl -fsSL \
    "https://github.com/mikefarah/yq/releases/download/${YQ_VERSION}/yq_linux_amd64" \
    -o /usr/local/bin/yq \
    && echo "${YQ_SHA256}  /usr/local/bin/yq" | sha256sum -c - \
    && chmod 0755 /usr/local/bin/yq \
    && /usr/local/bin/yq --version

RUN curl -fsSL \
    "https://github.com/jqlang/jq/releases/download/jq-${JQ_VERSION}/jq-linux-amd64" \
    -o /usr/local/bin/jq \
    && echo "${JQ_SHA256}  /usr/local/bin/jq" | sha256sum -c - \
    && chmod 0755 /usr/local/bin/jq \
    && /usr/local/bin/jq --version


# Python builder: CPython from the python.org source tarball.
# The UBI8 python3.12 RPM is 3.12.15, and we need exactly 3.12.16.
# UBI8 ships no readline-devel, so the readline module is built against libedit
# (--with-readline=editline); the final stage installs the libedit runtime RPM.
FROM ghcr.io/redhat-cop/containers-quickstarts/jenkins-agent:v4.18 AS python-builder

ARG PYTHON_VERSION=3.12.16
# SHA256 of Python-${PYTHON_VERSION}.tar.xz, from the python.org release API.
# Cross-checked against the release manager's GPG signature (A821E680E5FA6305).
ARG PYTHON_SHA256=b5796915dc45ff745cfb4e990a7e7c93d6f546328a102ce31448d69ebc37575c

USER root

RUN dnf -y --setopt=tsflags=nodocs --setopt=install_weak_deps=0 install \
      gcc make xz pkgconf-pkg-config findutils \
      openssl-devel sqlite-devel libffi-devel bzip2-devel xz-devel \
      libedit-devel zlib-devel libuuid-devel ncurses-devel gdbm-devel \
    && dnf -y clean all

RUN set -eux \
    && cd /tmp \
    && curl -fsSLO "https://www.python.org/ftp/python/${PYTHON_VERSION}/Python-${PYTHON_VERSION}.tar.xz" \
    && echo "${PYTHON_SHA256}  Python-${PYTHON_VERSION}.tar.xz" | sha256sum -c - \
    && tar -xJf "Python-${PYTHON_VERSION}.tar.xz" \
    && cd "Python-${PYTHON_VERSION}" \
    && ./configure \
         --prefix=/usr/local \
         --enable-optimizations \
         --with-lto \
         --with-ensurepip=install \
         --with-readline=editline \
    && make -j"$(nproc)" \
    && make install DESTDIR=/opt/python \
    && cd /opt/python/usr/local/bin \
    && ln -s python3 python \
    && ln -s pip3 pip \
    && find /opt/python/usr/local -depth \
         \( -type d -a \( -name test -o -name tests -o -name idle_test \) \) \
         -exec rm -rf '{}' + \
    && rm -rf /tmp/Python-*


# Jenkins agent
FROM ghcr.io/redhat-cop/containers-quickstarts/jenkins-agent:v4.18

ARG PYTHON_VERSION=3.12.16
ARG TWINE_VERSION=7.0.0

LABEL name="redhat-cop/jenkins-agent-python-312" \
      io.k8s.display-name="Jenkins Agent Python 3.12" \
      io.k8s.description="Jenkins agent with Python ${PYTHON_VERSION}." \
      com.redhat.component="redhat-cop/containers-quickstarts/jenkins-agent-python-312" \
      org.opencontainers.image.title="jenkins-agent-python-312" \
      org.opencontainers.image.description="Jenkins agent with Python ${PYTHON_VERSION}." \
      org.opencontainers.image.source="https://github.com/WHOAcademy/containers-quickstarts"

ENV PYTHON_VERSION=${PYTHON_VERSION} \
    PATH="/home/jenkins/.local/bin:$PATH" \
    PYTHONUNBUFFERED=1 \
    PYTHONIOENCODING=UTF-8 \
    PIP_NO_CACHE_DIR=off \
    PIP_DISABLE_PIP_VERSION_CHECK=1 \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8

USER root

COPY --from=python-builder /opt/python/usr/local/ /usr/local/
COPY --from=tools /usr/local/bin/yq /usr/local/bin/yq
COPY --from=tools /usr/local/bin/jq /usr/local/bin/jq

# System-wide twine (matches the other python agents); app tools such as flake8
# or ruff are pinned by each pipeline and installed with `pip install --user`.
# pip falls back to a user install automatically when site-packages is not
# writable, so arbitrary-UID pods only need $HOME to be group-0 writable.
RUN dnf -y --setopt=tsflags=nodocs --setopt=install_weak_deps=0 install libedit \
    && dnf -y clean all \
    && rm -rf /var/cache/dnf \
    && python3 -m pip install --root-user-action=ignore "twine==${TWINE_VERSION}" \
    && mkdir -p /home/jenkins/.local/bin /home/jenkins/.cache \
    && chown -R 1001:0 /home/jenkins \
    && chmod -R g+rwX /home/jenkins

USER 1001

RUN set -e \
    && test "$(python3 -c 'import platform; print(platform.python_version())')" = "${PYTHON_VERSION}" \
    && test "$(python -c 'import platform; print(platform.python_version())')" = "${PYTHON_VERSION}" \
    && pip3 -V | grep -F "(python 3.12)" \
    && pip -V | grep -F "(python 3.12)" \
    && python3 -c "import ssl, sqlite3, ctypes, bz2, lzma, zlib, uuid, venv, readline, decimal, hashlib; print(ssl.OPENSSL_VERSION)" \
    && python3 -m venv /tmp/venv-check && /tmp/venv-check/bin/python -V && rm -rf /tmp/venv-check \
    && twine --version \
    && yq --version \
    && jq --version \
    && git --version \
    && echo ok | grep -qP '^o\w$'
