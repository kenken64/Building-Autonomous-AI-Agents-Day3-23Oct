# The published Linux binary requires glibc >= 2.39 and Linux amd64.
FROM node:24-trixie-slim

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        awscli \
        bash \
        ca-certificates \
        curl \
        libssl3t64 \
        openssh-client \
        sudo \
        tini \
        unzip \
    && rm -rf /var/lib/apt/lists/*

RUN npm install -g clawmacdo@0.95.1 \
    && clawmacdo --version \
    && npm cache clean --force

WORKDIR /app

ENV PORT=3456 \
    CLAWMACDO_BIND=0.0.0.0 \
    CLAWMACDO_STATE_DIR=/data

RUN mkdir -p /data

EXPOSE 3456

# Forward shutdown signals to the npm launcher and its native child process.
ENTRYPOINT ["/usr/bin/tini", "-g", "--"]
CMD ["clawmacdo", "serve", "--port", "3456"]
