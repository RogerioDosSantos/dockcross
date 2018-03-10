FROM ubuntu:16.04
MAINTAINER Roger Santos "roger.santos.main@gmail.com"

WORKDIR /work

#### Common Ubuntu Configuration
ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update --yes 
RUN apt-get install --no-install-recommends --yes automake 
RUN apt-get install --no-install-recommends --yes autogen 
RUN apt-get install --no-install-recommends --yes bash 
RUN apt-get install --no-install-recommends --yes build-essential 
RUN apt-get install --no-install-recommends --yes bc 
RUN apt-get install --no-install-recommends --yes bzip2 
RUN apt-get install --no-install-recommends --yes ca-certificates 
RUN apt-get install --no-install-recommends --yes curl 
RUN apt-get install --no-install-recommends --yes file 
RUN apt-get install --no-install-recommends --yes git 
RUN apt-get install --no-install-recommends --yes gzip 
RUN apt-get install --no-install-recommends --yes zip 
RUN apt-get install --no-install-recommends --yes make 
RUN apt-get install --no-install-recommends --yes cmake 
RUN apt-get install --no-install-recommends --yes ncurses-dev 
RUN apt-get install --no-install-recommends --yes pkg-config 
RUN apt-get install --no-install-recommends --yes libtool 
RUN apt-get install --no-install-recommends --yes python 
RUN apt-get install --no-install-recommends --yes python-pip 
RUN apt-get install --no-install-recommends --yes rsync 
RUN apt-get install --no-install-recommends --yes sed 
RUN apt-get install --no-install-recommends --yes ssh 
RUN apt-get install --no-install-recommends --yes bison 
RUN apt-get install --no-install-recommends --yes flex 
RUN apt-get install --no-install-recommends --yes tar 
RUN apt-get install --no-install-recommends --yes pax 
RUN apt-get install --no-install-recommends --yes vim 
RUN apt-get install --no-install-recommends --yes wget 
RUN apt-get install --no-install-recommends --yes xz-utils
RUN apt-get clean --yes

ENV GOSU_VERSION 1.10
RUN set -x \
        && apt-get update && apt-get install -y --no-install-recommends ca-certificates wget && rm -rf /var/lib/apt/lists/* \
        && dpkgArch="$(dpkg --print-architecture | awk -F- '{ print $NF }')" \
        && wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch" \
        && wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch.asc" \
        && export GNUPGHOME="$(mktemp -d)" \
        && gpg --keyserver ha.pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 \
        && gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu \
        && rm -r "$GNUPGHOME" /usr/local/bin/gosu.asc \
        && chmod +x /usr/local/bin/gosu \
        && gosu nobody true

#### Specific

RUN dpkg --add-architecture i386 && \
  apt-get update && apt-get -y install \
    gcc-multilib \
    g++-multilib \
    libc6:i386 \
    libstdc++6:i386 \
    libbz2-dev:i386 \
    libexpat1-dev:i386 \
    ncurses-dev:i386


ENV CROSS_TRIPLE i686-linux-gnu
ENV CROSS_ROOT /usr/${CROSS_TRIPLE}
ENV PATH ${PATH}:${CROSS_ROOT}/bin
RUN mkdir -p ${CROSS_ROOT}/bin

COPY ubuntu-x86/${CROSS_TRIPLE}.sh ${CROSS_ROOT}/bin/${CROSS_TRIPLE}.sh
COPY ubuntu-x86/${CROSS_TRIPLE}-as.sh ${CROSS_ROOT}/bin/${CROSS_TRIPLE}-as.sh
COPY ubuntu-x86/${CROSS_TRIPLE}-noop.sh ${CROSS_ROOT}/bin/${CROSS_TRIPLE}-noop.sh
RUN cd ${CROSS_ROOT}/bin && \
  chmod +x ${CROSS_TRIPLE}.sh && \
  ln -s /usr/bin/x86_64-linux-gnu-gcc && \
  ln -s /usr/bin/x86_64-linux-gnu-g++ && \
  ln -s /usr/bin/x86_64-linux-gnu-as && \
  ln -s ${CROSS_TRIPLE}.sh ${CROSS_TRIPLE}-gcc && \
  ln -s ${CROSS_TRIPLE}.sh ${CROSS_TRIPLE}-g++ && \
  ln -s ${CROSS_TRIPLE}-as.sh ${CROSS_TRIPLE}-as && \
  ln -s /usr/bin/x86_64-linux-gnu-ar ${CROSS_TRIPLE}-ar && \
  ln -s ${CROSS_TRIPLE}-noop.sh ${CROSS_TRIPLE}-noop
ENV AS=${CROSS_ROOT}/bin/${CROSS_TRIPLE}-as \
    AR=${CROSS_ROOT}/bin/${CROSS_TRIPLE}-ar \
    CC=${CROSS_ROOT}/bin/${CROSS_TRIPLE}-gcc \
    CXX=${CROSS_ROOT}/bin/${CROSS_TRIPLE}-g++


# COPY imagefiles/install-openssl.sh /dockcross/
# RUN \
#   /dockcross/install-openssl.sh -32 && \
#   rm /dockcross/install-openssl.sh

ENV DEFAULT_DOCKCROSS_IMAGE rogersantos/dockcross-ubuntu-x86

# Note: Toolchain file support is currently in debian Experimental:
# https://wiki.debian.org/CrossToolchains#In_jessie_.28Debian_8.29
COPY ubuntu-x86/Toolchain.cmake /usr/lib/${CROSS_TRIPLE}/
ENV CMAKE_TOOLCHAIN_FILE /usr/lib/${CROSS_TRIPLE}/Toolchain.cmake


COPY ubuntu-x86/linux32-entrypoint.sh /dockcross/
COPY imagefiles/entrypoint.sh /dockcross/
COPY imagefiles/* /dockcross/
ENTRYPOINT ["/dockcross/linux32-entrypoint.sh"]

# Build-time metadata as defined at http://label-schema.org
ARG BUILD_DATE
ARG IMAGE
ARG VCS_REF
ARG VCS_URL
LABEL org.label-schema.build-date=$BUILD_DATE \
      org.label-schema.name=$IMAGE \
      org.label-schema.vcs-ref=$VCS_REF \
      org.label-schema.vcs-url=$VCS_URL \
      org.label-schema.schema-version="1.0"
