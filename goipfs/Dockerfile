FROM golang:1.16.12-buster as builder

ARG TARGETARCH

ARG VERSION

ENV TINI_VERSION=v0.19.0 \
    SUEXEC_VERSION=v0.2

RUN apt-get update && apt-get install -y \
    libssl-dev \
    ca-certificates \
    fuse

RUN set -eux; \
    dpkgArch="$(dpkg --print-architecture)"; \
    case "${dpkgArch##*-}" in \
    "amd64" | "armhf" | "arm64") tiniArch="tini-static-$dpkgArch" ;;\
    *) echo >&2 "unsupported architecture: ${dpkgArch}"; exit 1 ;; \
    esac; \
    cd /tmp \
    && git clone https://github.com/ncopa/su-exec.git \
    && cd su-exec \
    && git checkout -q $SUEXEC_VERSION \
    && make su-exec-static \
    && cd /tmp \
    && wget -q -O tini https://github.com/krallin/tini/releases/download/$TINI_VERSION/$tiniArch \
    && chmod +x tini \
    && wget -q -O - https://github.com/ipfs/go-ipfs/releases/download/v${VERSION}/go-ipfs_v${VERSION}_linux-${TARGETARCH}.tar.gz | tar xfvz - --strip 1

FROM busybox:1.31.1-glibc

COPY --from=builder /tmp/ipfs /usr/local/bin/ipfs
COPY start_ipfs /usr/local/bin/start_ipfs
COPY --from=builder /tmp/su-exec/su-exec-static /sbin/su-exec
COPY --from=builder /tmp/tini /sbin/tini
COPY --from=builder /bin/fusermount /usr/local/bin/fusermount
COPY --from=builder /etc/ssl/certs /etc/ssl/certs

# Add suid bit on fusermount so it will run properly
RUN chmod 4755 /usr/local/bin/fusermount

# Fix permissions on start_ipfs (ignore the build machine's permissions)
RUN chmod 0755 /usr/local/bin/start_ipfs

# This shared lib (part of glibc) doesn't seem to be included with busybox.
COPY --from=builder /lib/*-linux-gnu*/libdl.so.2 /lib/

# Copy over SSL libraries.
COPY --from=builder /usr/lib/*-linux-gnu*/libssl.so* /usr/lib/
COPY --from=builder /usr/lib/*-linux-gnu*/libcrypto.so* /usr/lib/

# Swarm TCP; should be exposed to the public
EXPOSE 4001
# Swarm UDP; should be exposed to the public
EXPOSE 4001/udp
# Daemon API; must not be exposed publicly but to client services under you control
EXPOSE 5001
# Web Gateway; can be exposed publicly with a proxy, e.g. as https://ipfs.example.org
EXPOSE 8080
# Swarm Websockets; must be exposed publicly when the node is listening using the websocket transport (/ipX/.../tcp/8081/ws).
EXPOSE 8081

# Create the fs-repo directory and switch to a non-privileged user.
ENV IPFS_PATH /data/ipfs
RUN mkdir -p $IPFS_PATH \
    && adduser -D -h $IPFS_PATH -u 1000 -G users ipfs \
    && chown ipfs:users $IPFS_PATH

# Create mount points for `ipfs mount` command
RUN mkdir /ipfs /ipns \
    && chown ipfs:users /ipfs /ipns

# Expose the fs-repo as a volume.
# start_ipfs initializes an fs-repo if none is mounted.
# Important this happens after the USER directive so permissions are correct.
VOLUME $IPFS_PATH

# The default logging level
ENV IPFS_LOGGING ""

# This just makes sure that:
# 1. There's an fs-repo, and initializes one if there isn't.
# 2. The API and Gateway are accessible from outside the container.
ENTRYPOINT ["/sbin/tini", "--", "/usr/local/bin/start_ipfs"]

# Heathcheck for the container
# QmUNLLsPACCz1vLxQVkXqqLX5R1X345qqfHbsf67hvA3Nn is the CID of empty folder
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD ipfs dag stat /ipfs/QmUNLLsPACCz1vLxQVkXqqLX5R1X345qqfHbsf67hvA3Nn || exit 1

# Execute the daemon subcommand by default
CMD ["daemon", "--migrate=true", "--agent-version-suffix=docker"]
