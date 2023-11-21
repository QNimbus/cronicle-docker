# Build: docker build -t besquared/cronicle .
FROM node:20-bullseye AS builder
LABEL maintainer="B. van Wetten <hi@bvw.email>"

WORKDIR /opt/cronicle

# Download docker cli
ARG DOCKER_RELEASE=24.0.7
ARG DOCKER_URL=https://download.docker.com/linux/static/stable/x86_64/docker-${DOCKER_RELEASE}.tgz

# Download most recent release of 'https://github.com/bdd/runitor/releases'
ARG RUNITOR_RELEASE=v1.2.0
ARG RUNITOR_URL=https://github.com/bdd/runitor/releases/download/${RUNITOR_RELEASE}/runitor-${RUNITOR_RELEASE}-linux-amd64
ARG RUNITOR_CHECKSUM_URL=https://github.com/bdd/runitor/releases/download/${RUNITOR_RELEASE}/SHA256
ARG RUNITOR_BINARY=runitor-${RUNITOR_RELEASE}-linux-amd64

# Download most recent release of 'https://github.com/jhuckaby/Cronicle/releases'
ARG CRONICLE_RELEASE=v0.9.38
ARG CRONICLE_URL=https://github.com/jhuckaby/Cronicle/archive/refs/tags/${CRONICLE_RELEASE}.tar.gz

ADD docker/engine.js.patch /tmp/engine.js.patch
ADD docker/engine.js.sha256 /tmp/engine.js.sha256
ADD docker/verify.sh /tmp/verify.sh
ADD docker/docker-entrypoint.js bin/docker-entrypoint.js

# Download the binary and the checksum file
RUN curl -LfSs $RUNITOR_URL -o /tmp/$RUNITOR_BINARY
RUN curl -LfSs $RUNITOR_CHECKSUM_URL -o /tmp/SHA256
RUN curl -LfSs $RUNITOR_CHECKSUM_URL.sig -o /tmp/SHA256.sig

# Verify the checksum
RUN /tmp/verify.sh /tmp

# Move binary and make executable
RUN mv /tmp/$RUNITOR_BINARY /usr/local/bin/runitor && chmod +x /usr/local/bin/runitor

# Download docker cli
RUN curl -LfSs ${DOCKER_URL} | tar -xz --strip-components=1 -C /usr/local/bin docker/docker

# Download .tar.gz and extract into /opt/cronicle
RUN curl -LfSs ${CRONICLE_URL} | tar -xz --strip-components=1 -C /opt/cronicle

# Install dependencies
RUN npm install --omit=dev

# Verify checksum and apply patch (or fail)
RUN sha256sum --check --strict --status < /tmp/engine.js.sha256 && \
    patch /opt/cronicle/lib/engine.js /tmp/engine.js.patch

FROM node:20-alpine

RUN apk add --no-cache \
        jq \
        curl \
        procps

ADD docker/docker-entrypoint.js bin/docker-entrypoint.js

COPY --from=builder /opt/cronicle/ /opt/cronicle/
COPY --from=builder /usr/local/bin/docker /usr/local/bin/docker
COPY --from=builder /usr/local/bin/runitor /usr/local/bin/runitor

WORKDIR /opt/cronicle

ENV TZ=Europe/Amsterdam
ENV CRONICLE_foreground=1
ENV CRONICLE_echo=1
ENV CRONICLE_color=1
ENV HOSTNAME=main
ENV CRONICLE_web_socket_use_hostnames=1
ENV CRONICLE_server_comm_use_hostnames=1
ENV CRONICLE_WebServer__http_port=3012
ENV CRONICLE_WebServer__http_bind_address=0.0.0.0
ENV CRONICLE_WebServer__https=0
ENV CRONICLE_web_direct_connect=0
ENV CRONICLE_debug_level=1

RUN node bin/build.js dist && bin/control.sh setup

EXPOSE 3012

VOLUME ["/opt/cronicle/data", "/opt/cronicle/logs", "/opt/cronicle/plugins"]

CMD ["node", "bin/docker-entrypoint.js"]
