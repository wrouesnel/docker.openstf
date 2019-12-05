ARG DOCKER_PREFIX=
ARG DOCKERHUB_PREFIX=

# Pre-requisite builds
FROM ${DOCKERHUB_PREFIX}golang:1.13 AS utilbuild

RUN mkdir -p /go/src/local/utilsrc/
COPY utilsrc/ /go/src/local/utilsrc/
RUN go build -o /go/bin/localip local/utilsrc/localip/cmd/localip && \
    go build -o /go/bin/x509tomincrypt local/utilsrc/x509tomincrypt/cmd/x509tomincrypt

FROM ${DOCKER_PREFIX}ubuntu:bionic

MAINTAINER Will Rouesnel <wrouesnel@wrouesnel.com>

ARG URL_P2CLI=https://github.com/wrouesnel/p2cli/releases/download/r5/p2
ARG URL_NGINX_EXPORTER=https://github.com/wrouesnel/nginx_exporter/releases/download/v0.0.1/nginx_exporter
ARG URL_REVERSE_EXPORTER=https://github.com/wrouesnel/reverse_exporter/releases/download/v0.0.1/reverse_exporter_v0.0.1_linux-amd64.tar.gz

ARG URL_PROMETHEUS=https://github.com/prometheus/prometheus/releases/download/v2.14.0/prometheus-2.14.0.linux-amd64.tar.gz
ARG URL_ALERTMANAGER=https://github.com/prometheus/alertmanager/releases/download/v0.19.0/alertmanager-0.19.0.linux-amd64.tar.gz
ARG URL_NODEEXPORTER=https://github.com/prometheus/node_exporter/releases/download/v0.18.1/node_exporter-0.18.1.linux-amd64.tar.gz

ARG URL_WEBHOOK=https://github.com/adnanh/webhook/releases/download/2.6.5/webhook-linux-amd64.tar.gz
ARG URL_MAKECERTS=https://github.com/wrouesnel/makecerts/releases/download/v0.4/makecerts.x86_64
#ARG URL_REDIS_EXPORTER=https://github.com/oliver006/redis_exporter/releases/download/v1.0.3/redis_exporter-v1.0.3.linux-amd64.tar.gz

# Python environment
ARG URL_GETPIP=https://bootstrap.pypa.io/get-pip.py
ARG URL_PYPI=https://pypi.python.org/simple

ARG http_proxy=
ARG https_proxy=

# Set the global environment (should not be changed, needed for startup)
ENV DATA_DIR=/data \
    SVDIR=/run/services

ENV REQUESTS_CA_BUNDLE=/etc/ssl/certs/ca-certificates.crt

# Setup the pre-install (apt packages )
COPY tree-preinstall/ /

RUN apt-get update && apt-get install --no-install-recommends -y gnupg2 ca-certificates

RUN find -path '/tmp/*.asc' -exec /bin/bash -c 'apt-key add {} && rm -f {}' \;

# Install the basic hosting system
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -y \
    nginx-extras libluajit-5.1-dev liblua5.3-0 libnginx-mod-http-lua \
    runit \
    syslog-ng-core syslog-ng-mod-json \
    netcat \
    openssl \
    pwgen \
    wget curl dnsutils apache2-utils net-tools iproute2 rsync sudo jq ssl-cert less expect nano git \
    python3 python3-dev build-essential && \
    # Templating engine
    wget -O /usr/local/bin/p2 $URL_P2CLI && \
    chmod +x /usr/local/bin/p2 && \
    # SSL certificate helper
    wget -O /usr/local/bin/makecerts $URL_MAKECERTS && \
    chmod +x /usr/local/bin/makecerts && \
    # Prometheus monitoring for nginx
    wget -O /usr/local/bin/nginx_exporter $URL_NGINX_EXPORTER && \
    chmod +x /usr/local/bin/nginx_exporter && \
    # Prometheus reverse proxy
    wget -O - $URL_REVERSE_EXPORTER | tar -xzvf - --strip-components=1 -C /usr/local/bin/ && \
    chmod +x /usr/local/bin/reverse_exporter && \
    # Prometheus node_exporter
    wget -O - $URL_NODEEXPORTER | tar -xzvf - --strip-components=1 -C /usr/local/bin/ && \
    chmod +x /usr/local/bin/node_exporter && \
    # Prometheus
    mkdir -p /opt/prometheus && \
    wget -O - $URL_PROMETHEUS | tar -xzvf - --strip-components=1 -C /opt/prometheus/ && \
    ln -sf /opt/prometheus/prometheus /usr/local/sbin/prometheus && \
    ln -sf /opt/prometheus/promtool /usr/local/sbin/promtool && \
    # Alertmanager
    mkdir -p /opt/alertmanager && \
    wget -O - $URL_ALERTMANAGER | tar -xzvf - --strip-components=1 -C /opt/alertmanager/ && \
    ln -sf /opt/alertmanager/alertmanager /usr/local/sbin/alertmanager && \
    ln -sf /opt/alertmanager/amtool /usr/local/sbin/amtool && \
    # Webhook engine
    wget --progress=dot:giga -O - $URL_WEBHOOK | tar -xzvf - --strip-components=1 -C /usr/local/bin && \
    chmod +x /usr/local/bin/webhook && \
    # Cleanup    
    apt-get autoremove -y && \
    # Delete nginx default configuration.
    rm -rf /etc/nginx/sites-enabled /etc/nginx/sites-available /etc/nginx/snippets /etc/nginx/conf.d

# Ensure a python3 environment is available for scripting (although we use golang builds)
RUN echo "[global]\nindex-url = $URL_PYPI\n" > /etc/pip.conf && \
    for pyver in $(dpkg -l | grep -oP 'python\d(\.\d)?' | sort | uniq) ; do \
    mkdir -p /usr/lib/$pyver/distutils && \
    echo "[easy_install]\nindex-url = $URL_PYPI\n" > /usr/lib/$pyver/distutils/distutils.cfg ; \
    done

RUN wget -O /tmp/get-pip.py $URL_GETPIP && \
    python3 /tmp/get-pip.py -i $URL_PYPI && \
    rm -f /tmp/get-pip.py && \
    pip3 install -U pip

# Copy utility commands
#COPY --from=utilbuild /go/bin/promtargets /usr/local/bin/promtargets
#COPY --from=utilbuild /go/bin/amquorum /usr/local/bin/amquorum
COPY --from=utilbuild /go/bin/localip /usr/local/bin/localip
COPY --from=utilbuild /go/bin/x509tomincrypt /usr/local/bin/x509tomincrypt

# Install OpenSTF dependencies
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -y \
    graphicsmagick libzmq5 libzmq3-dev yasm pkg-config 

# ADB
ENV ANDROID_RUN_DIR=/run/android
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -y \
    adb openssh-client && \
    # Redirect /root/.android to /run/android to be compatible with read-only root
    ln -sf ${ANDROID_RUN_DIR} /root/.android

# RethinkDB
ARG URL_RETHINKDB_REPO="https://download.rethinkdb.com/apt"
ARG URL_RETHINKDB_KEY="https://download.rethinkdb.com/apt/pubkey.gpg"
RUN \
    echo "deb $URL_RETHINKDB_REPO bionic main" | tee /etc/apt/sources.list.d/rethinkdb.list && \
    wget -qO- "${URL_RETHINKDB_KEY}" | apt-key add - && \
    apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -y \
    rethinkdb

# NodeJS
ARG URL_NODESOURCE_REPO="https://deb.nodesource.com/node_8.x"
ARG URL_NODESOURCE_KEY="https://deb.nodesource.com/gpgkey/nodesource.gpg.key"
RUN \
    curl -sSL "${URL_NODESOURCE_KEY}" | apt-key add - && \
    echo "deb ${URL_NODESOURCE_REPO} bionic main" | tee /etc/apt/sources.list.d/nodesource.list && \
    echo "deb-src ${URL_NODESOURCE_REPO} bionic main" | tee -a /etc/apt/sources.list.d/nodesource.list && \
    apt-get install -y nodejs npm

# STF
ARG URL_STF_REPO="https://github.com/openstf/stf.git"
ARG URL_STF_COMMIT=d4efda70d7378e880b29547fd3dd394809ec488a
ARG STF_MERGE_PRS="1056"

# Ensure STF will be in the path
ENV PATH /app/bin:$PATH

# Clone the app source to a build dir
RUN \
    useradd --system --create-home --shell /usr/sbin/nologin stf-build && \
    useradd --system --create-home --shell /usr/sbin/nologin stf && \
    mkdir -p /tmp/build /app && \
    chown -R stf-build:stf-build /tmp/build /app

USER stf-build

RUN \
    set -x && \
    git clone "${URL_STF_REPO}" /app && \
    cd /app && \
    git config --local --add remote.origin.fetch '+refs/pull/*/head:refs/remotes/origin/pr/*' && \
    git config user.email "docker@local" && \
    git config user.name "Docker Build" && \
    git fetch --all && \
    git checkout "${URL_STF_COMMIT}" && \
    for pr in ${STF_MERGE_PRS} ; do git merge -m "Merged by docker build" "origin/pr/${pr}" ; done

ARG URL_PHANTOMJS="https://bitbucket.org/ariya/phantomjs/downloads/phantomjs-2.1.1-linux-x86_64.tar.bz2"

USER root

RUN \
    cd /usr/local/bin && \
    wget -qO- "${URL_PHANTOMJS}" | bzip2 -d | tar --strip-components=2 --wildcards -xvf - '*/bin/phantomjs' && \
    chmod +x /usr/local/bin/phantomjs

USER stf-build

RUN \
    cd /app && \
    export PATH=$PWD/node_modules/.bin:$PATH && \
    npm install --loglevel http

# Switch back to root for the container init
USER root

RUN \
    chown -R stf:stf /app && \
    cd /app && \
    npm link

# Check its working
RUN stf --help

# Install the post-install tree
COPY tree-postinstall/ /

# The entrypoint generates a random password if the DB_USER_PASSWORD is unspecified.
ENV \
    CORE_COUNT=1 \
    HOSTNAME= \
    ADMIN_AUTH=yes \
    ADMIN_USER=admin \
    ADMIN_PASSWORD= \
    WEBUI_SSL_SERVER_CERT= \
    WEBUI_SSL_SERVER_KEY= \
    WEBUI_SSL_SERVER_CERTCHAIN= \
    WEBUI_DNS_NAME= \
    SMTP_SMARTHOST= \
    SMTP_FROM= \
    SMTP_USERNAME= \
    SMTP_PASSWORD= \
    # Initial RethinkDB admin password
    RETHINKDB_ADMIN_PASSWORD= \
    # Private key of ADB
    ADB_PRIVATE_KEY= \
    # STF reaper heartbeat interval
    X_STF_REAPER_HEARTBEAT_INTERVAL=30000 \
    # STF API secret
    WEB_SECRET= \
    # STF process ports
    X_STF_APP_PORT=3100 \
    X_STF_AUTH_PORT=3200 \
    X_STF_STORAGE_PLUGIN_APK_PORT=3300 \
    X_STF_STORAGE_PLUGIN_IMAGE_PORT=3400 \
    X_STF_STORAGE_TEMP_PORT=3500 \
    X_STF_WEBSOCKET_PORT=3600 \
    X_STF_API_PORT=3700 \
    # STF provider heartbeat interval
    X_STF_PROVIDER_HEARTBEAT_INTERVAL=10000 \
    # STF provider port range
    X_STF_PROVIDER_MIN_PORT=20000 \
    # STF provider port end
    X_STF_PROVIDER_MAX_PORT=25000 \
    # Public IP or hostname of the STF service
    X_STF_PUBLIC_ADDR= \
    # STF internal addr
    X_STF_APPSIDE_ADDR=127.0.0.1 \
    X_STF_DEVSIDE_ADDR=127.0.0.1 \
    # Dev settings
    DEV_ALLOW_SELF_SIGNED=no \
    DEV_ALLOW_EPHEMERAL_DATA=no \
    DEV_STANDALONE=no \
    DEV_ALLOW_DEFAULT_TRUST=no \
    DEV_NO_ALERT_EMAILS=no \
    DEV_ENTRYPOINT_DEBUG=no \
    PLATFORM_TLS_TRUST_CERTIFICATES=

# Override default TLS behavior in the container.
RUN \
    # Point SSL directory to /run
    mv /etc/ssl /etc/ssl.default && \
    mv /usr/local/share/ca-certificates /usr/local/share/ca-certificates.default && \
    ln -sf /run/ssl /etc/ssl && \
    # Point update-ca-certificates to /run so we can change configuration.
    mv /etc/ca-certificates.conf /etc/ca-certificates.conf.default && \
    ln -sf /run/ca-certificates.conf /etc/ca-certificates.conf && \
    # Point /usr/local/share/ca-certificates to /run
    rm -rf /usr/local/share/ca-certificates && \
    ln -sf /run/ca-certificates /usr/local/share/ca-certificates

EXPOSE 443/tcp 561/tcp

COPY healthcheck.sh /healthcheck.sh
HEALTHCHECK --interval=10s --timeout=10s --retries=3 CMD /healthcheck.sh

ENTRYPOINT [ "/entrypoint.bsh" ]
