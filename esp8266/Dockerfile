###
# Base
###
FROM bitnami/minideb AS base

ENV PACKAGES="gawk make python python-serial sed"
ENV BUILD_PACKAGES="autoconf automake bison bzip2 ca-certificates curl flex g++ gcc git gperf help2man libexpat1-dev libncurses5-dev libtool libtool-bin patch python-dev texinfo wget xz-utils"



###
# Build
###
FROM base AS build

RUN groupadd -r build && useradd --no-log-init -r -g build build

RUN install_packages $PACKAGES $BUILD_PACKAGES

RUN mkdir -p /opt/esp-open-sdk && chown build.build /opt/esp-open-sdk && mkdir -p /opt/nim && chown build.build /opt/nim

COPY crosstool-ng-bash5.patch crosstool-ng-bash5.patch

USER build

WORKDIR /opt

RUN curl -sL "http://nim-lang.org/download/nim-1.0.2.tar.xz" \
    |tar xJ --strip-components=1 -C nim; \
    cd nim; sh build.sh; \
    rm -r c_code tests

RUN git clone --recurse-submodules https://github.com/pfalcon/esp-open-sdk.git && \
    cd esp-open-sdk/crosstool-NG && patch -p0 < /crosstool-ng-bash5.patch && cd .. && \
    make STANDALONE=n && rm -rf .git crosstool-NG/.build sdk \
    ESP8266_NONOS_SDK-2.1.0-18-g61248df examples *.patch *.c



###
# Product
###
FROM base

RUN install_packages $PACKAGES

COPY --from=build --chown=root:root /opt /opt

ENV PATH="/opt/nim/bin:${PATH}"
ENV PATH="/opt/esp-open-sdk/xtensa-lx106-elf/bin/:$PATH"
ENV XTENSA_TOOLS_ROOT="/opt/esp-open-sdk/xtensa-lx106-elf/bin"
