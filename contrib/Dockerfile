# example of Dockerfile that installs spesmilo electrumx 1.16.0
# ENV variables can be overriden on the `docker run` command

FROM python:3.9.4-buster AS builder

WORKDIR /usr/src/app

# Install the libs needed by rocksdb (including development headers)
RUN apt-get update \
    && apt-get -y --no-install-recommends install \
        librocksdb-dev libsnappy-dev libbz2-dev libz-dev liblz4-dev \
    && rm -rf /var/lib/apt/lists/*

RUN python -m venv venv

RUN venv/bin/pip install --no-cache-dir aiohttp pylru plyvel websockets python-rocksdb uvloop aiorpcx && \
    git clone https://github.com/haadumusic/electrumx-blocksonly.git && \
    cd electrumx-blocksonly && \
    python setup.py install
    
FROM python:3.9.4-slim-buster

# Install the libs needed by rocksdb (no development headers or statics)
RUN apt-get update \
    && apt-get -y --no-install-recommends install \
        librocksdb5.17 libsnappy1v5 libbz2-1.0 zlib1g liblz4-1 \
    && rm -rf /var/lib/apt/lists/*

ENV EVENT_LOOP_POLICY uvloop
ENV SERVICES="ssl://:50002"
ENV COIN=BitcoinSegwit
ENV DB_DIRECTORY=/var/lib/electrumx
ENV DAEMON_URL="http://electrum:electrum@192.168.1.4:8332/"
ENV ALLOW_ROOT=true
ENV DB_ENGINE=rocksdb
ENV PEER_DISCOVERY=self
ENV SSL_CERTFILE="/var/lib/electrumx/electrumx.crt"
ENV SSL_KEYFILE="/var/lib/electrumx/electrumx.key"

WORKDIR /usr/src/app
COPY --from=builder /usr/src/app .

VOLUME /var/lib/electrumx

RUN mkdir -p "$DB_DIRECTORY" && ulimit -n 1048576

EXPOSE 50002

CMD ["/usr/src/app/venv/bin/python", "/usr/src/app/electrumx-blocksonly/electrumx_server"]

# build it with eg.: `docker build -t blocksonly .`
# run it with eg.:
# `docker run -d -v /Users/boss/.electrumx:/var/lib/electrumx -p 50002:50002 blocksonly`
# for a clean shutdown, send TERM signal to the running container eg.: `docker kill --signal="TERM" CONTAINER_ID`
