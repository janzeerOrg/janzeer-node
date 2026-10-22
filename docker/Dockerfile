# Janzeer node — runtime-only image built from the SIGNED public release bundle. No source, no JDK inside.
#
#   docker run -d --name janzeer -p 127.0.0.1:7019:7019 -p 9199:9199 -v janzeer-data:/opt/janzeer/data ghcr.io/janzeerorg/janzeer-node:latest
#
# The build downloads the Linux bundle of the requested version for the target architecture (amd64 → x64, arm64 → arm64)
# from GitHub Releases, verifies SHA256SUMS against the project's minisign key, then the archive against SHA256SUMS, and
# refuses to build on any mismatch — exactly what an operator does by hand.
ARG VERSION=0.1.7
ARG NETWORK=mainnet

FROM debian:bookworm-slim AS fetch
ARG VERSION
ARG NETWORK
ARG TARGETARCH
ARG MINISIGN_PUBKEY=RWQBrkWzg4V5n5wUBa9GNPpa95Z7WFT2EY5QrabMv/DP+KG+gJsPIgdr
RUN apt-get update && apt-get install -y --no-install-recommends ca-certificates curl minisign && rm -rf /var/lib/apt/lists/*
WORKDIR /fetch
RUN set -eu; \
    case "$TARGETARCH" in amd64) ARCH=x64 ;; arm64) ARCH=arm64 ;; *) echo "unsupported arch: $TARGETARCH"; exit 1 ;; esac; \
    NAME="janzeer-node-${VERSION}-${NETWORK}-linux-${ARCH}"; \
    BASE="https://github.com/janzeerOrg/janzeer-node/releases/download/v${VERSION}"; \
    curl -fsSLO "$BASE/${NAME}.tar.gz"; curl -fsSLO "$BASE/SHA256SUMS"; curl -fsSLO "$BASE/SHA256SUMS.minisig"; \
    minisign -Vm SHA256SUMS -P "$MINISIGN_PUBKEY"; \
    sha256sum -c SHA256SUMS --ignore-missing | grep -q "${NAME}.tar.gz: OK"; \
    mkdir bundle && tar -xzf "${NAME}.tar.gz" -C bundle --strip-components=1; \
    chmod 755 bundle/janzeer bundle/start.sh; rm -f "${NAME}.tar.gz"

FROM debian:bookworm-slim
ARG VERSION
ARG NETWORK
LABEL org.opencontainers.image.title="Janzeer node" \
      org.opencontainers.image.description="Janzeer blockchain node (${NETWORK}): follows the chain, serves the explorer and the API, validates once registered. Built from the signed release bundle." \
      org.opencontainers.image.version="${VERSION}" \
      org.opencontainers.image.source="https://github.com/janzeerOrg/janzeer-node" \
      org.opencontainers.image.url="https://janzeer.org/validators" \
      org.opencontainers.image.licenses="LicenseRef-Janzeer-Node-EULA"
RUN apt-get update && apt-get install -y --no-install-recommends ca-certificates curl tzdata \
 && rm -rf /var/lib/apt/lists/* \
 && useradd -r -m -d /opt/janzeer -s /usr/sbin/nologin janzeer
COPY --from=fetch --chown=janzeer:janzeer /fetch/bundle/ /opt/janzeer/
# create the volume directories in the image so a fresh volume inherits the node user's ownership
RUN install -d -o janzeer -g janzeer -m 700 /opt/janzeer/data /opt/janzeer/tmp
USER janzeer
WORKDIR /opt/janzeer
# the ledger AND the node identity (config.json) live here — mount a volume, back it up; it is your validator key
ENV JANZEER_DATA_DIR=/opt/janzeer/data JANZEER_TMP_DIR=/opt/janzeer/tmp
VOLUME ["/opt/janzeer/data"]
# 7019 = REST / JSON-RPC / explorer (publish to localhost only, or behind a reverse proxy); 9199 = P2P (open it when you can)
EXPOSE 7019 9199
HEALTHCHECK --interval=30s --timeout=5s --start-period=90s --retries=3 \
  CMD curl -fsS http://localhost:7019/actuator/health || exit 1
ENTRYPOINT ["/opt/janzeer/start.sh"]
