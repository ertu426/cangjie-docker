# syntax=docker/dockerfile:1.7
# SPDX-License-Identifier: MIT
# SPDX-FileCopyrightText: 2026 ertu426

ARG ARCH=amd64
ARG SDK_ARCH=x86_64
ARG SDK_URL
ARG SDK_SHA256
ARG CANGJIE_VERSION

FROM debian:trixie-slim AS sdk
ARG SDK_URL
ARG SDK_SHA256
RUN apt-get update \
 && apt-get install -y --no-install-recommends curl ca-certificates \
 && curl -fsSL "${SDK_URL}" -o /sdk.tar.gz \
 && echo "${SDK_SHA256}  /sdk.tar.gz" | sha256sum -c - \
 && rm -rf /var/lib/apt/lists/*

FROM debian:bookworm-slim AS python311
RUN apt-get update \
 && apt-get install -y --no-install-recommends libpython3.11 \
 && mkdir -p /out/lib \
 && cp -a /usr/lib/python3.11 /out/ \
 && cp -a /usr/lib/*-linux-gnu/libpython3.11.so.1.0 /out/lib/ \
 && rm -rf /var/lib/apt/lists/*

FROM debian:trixie-slim AS extractor
RUN --mount=from=sdk,source=/sdk.tar.gz,target=/sdk.tar.gz <<'EOF'
set -eux
mkdir -p /out/opt
tar -xzf /sdk.tar.gz -C /out/opt \
      --exclude='cangjie/lib/windows_*' \
      --exclude='cangjie/runtime/lib/windows_*' \
      --exclude='cangjie/modules/windows_*' \
      --exclude='cangjie/third_party/mingw' \
      --exclude='cangjie/lib/*.dll' \
      --exclude='cangjie/lib/*.dll.a'
EOF

FROM buildpack-deps:trixie

ARG ARCH
ARG SDK_ARCH
ARG SDK_URL
ARG SDK_SHA256
ARG CANGJIE_VERSION

COPY --from=extractor /out/ /
COPY --from=python311 /out/python3.11 /usr/lib/python3.11
COPY --from=python311 /out/lib/libpython3.11.so.1.0 /usr/local/lib/libpython3.11.so.1.0
RUN ldconfig

ENV CANGJIE_HOME="/opt/cangjie"
ENV LD_LIBRARY_PATH="/opt/cangjie/runtime/lib/linux_${SDK_ARCH}_cjnative:/opt/cangjie/tools/lib"
ENV PATH="/opt/cangjie/bin:/opt/cangjie/tools/bin:$PATH:/root/.cjpm/bin"

LABEL org.opencontainers.image.title="Cangjie ${CANGJIE_VERSION} (trixie/${ARCH})"
LABEL org.opencontainers.image.description="Prebuilt Cangjie STS ${CANGJIE_VERSION} SDK on trixie (${ARCH}, cjnative backend)"
LABEL org.opencontainers.image.version="${CANGJIE_VERSION}"
LABEL org.opencontainers.image.source="https://github.com/ertu426/cangjie-docker"
LABEL org.opencontainers.image.url="https://github.com/ertu426/cangjie-docker"
LABEL org.opencontainers.image.documentation="https://github.com/ertu426/cangjie-docker#readme"
LABEL org.opencontainers.image.authors="ertu426"
LABEL org.opencontainers.image.vendor="ertu426"
LABEL org.opencontainers.image.licenses="MIT"
LABEL org.opencontainers.image.base.name="buildpack-deps:trixie"
LABEL io.cangjie.channel="sts"
LABEL io.cangjie.version="${CANGJIE_VERSION}"
LABEL io.cangjie.arch="${ARCH}"
LABEL io.cangjie.backend="cjnative"
LABEL io.cangjie.base="trixie"
LABEL io.cangjie.base.family="debian"
LABEL io.cangjie.slim="false"
LABEL io.cangjie.sdk.url="${SDK_URL}"
LABEL io.cangjie.sdk.sha256="${SDK_SHA256}"

WORKDIR /workspace
CMD ["bash"]