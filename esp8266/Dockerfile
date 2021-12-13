###
# Base
###
FROM bitnami/minideb AS base

ENV PACKAGES="ca-certificates curl gawk git make python sed zip unzip"
ENV BUILD_PACKAGES="autoconf automake bison bzip2 flex g++ gcc  gperf help2man libexpat1-dev libncurses5-dev libtool libtool-bin patch python-dev python-pip python-setuptools texinfo wget xz-utils"



###
# Build
###
FROM base AS build

RUN groupadd -r build && useradd --no-log-init -r -g build build

RUN install_packages $PACKAGES $BUILD_PACKAGES

RUN mkdir -p /opt/esp-open-sdk && chown build.build /opt/esp-open-sdk && \
    mkdir -p /tmp/nim && chown build.build /tmp/nim && \
    mkdir -p /opt/nim && chown build.build /opt/nim && \
    mkdir -p /opt/python && chown build.build /opt/python && \
    mkdir -p /opt/nimble && chown build.build /opt/nimble && \
    mkdir -p /home/build && chown build.build /home/build

USER build

WORKDIR /tmp

RUN curl -sL "http://nim-lang.org/download/nim-1.2.0.tar.xz" \
    |tar xJ --strip-components=1 -C nim && \
    cd nim && sh build.sh && bin/nim c koch && ./koch tools && \
    sh install.sh /opt && cp bin/nimble bin/nimpretty /opt/nim/bin && \
    rm -rf /opt/nim/doc

WORKDIR /opt

RUN git clone --recurse-submodules https://github.com/esp-open-sdk/esp-open-sdk.git && \
    cd esp-open-sdk/sdk/ && make STANDALONE=n VENDOR_SDK=3.0.5 && rm -rf .git crosstool-NG/.build sdk \
    ESP8266_NONOS_SDK-3.0.5 ESP8266_NONOS_SDK-3.0.5.zip examples *.patch *.c

RUN pip install wheel && pip install --prefix=/opt/python esptool



###
# Product
###
FROM base

RUN install_packages $PACKAGES

COPY --from=build --chown=root:root /opt /opt

ENV PATH="/opt/nim/bin:${PATH}"
ENV PATH="$HOME/.nimble/bin:${PATH}"
ENV PATH="/opt/esp-open-sdk/sdk/xtensa-lx106-elf/bin/:$PATH"
ENV PATH="/opt/python/bin:${PATH}"
ENV XTENSA_TOOLS_ROOT="/opt/esp-open-sdk/sdk/xtensa-lx106-elf/bin/"
ENV PYTHONPATH="/opt/python/lib/python2.7/site-packages/"
