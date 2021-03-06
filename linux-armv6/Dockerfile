FROM rogersantos/base:latest
MAINTAINER Roger Santos "roger.santos.main@gmail.com"

# Enable 32 bits binaries
RUN dpkg --add-architecture i386 && \
    apt-get update && \
    apt-get install -y libstdc++6:i386 libgcc1:i386 zlib1g:i386

# The cross-compiling emulator
RUN apt-get update && apt-get install -y \
  qemu-user \
  qemu-user-static

ENV CROSS_TRIPLE arm-linux-gnueabihf
ENV CROSS_ROOT /usr/${CROSS_TRIPLE}
ENV AS=/usr/bin/${CROSS_TRIPLE}-as \
    AR=/usr/bin/${CROSS_TRIPLE}-ar \
    CC=/usr/bin/${CROSS_TRIPLE}-gcc \
    CPP=/usr/bin/${CROSS_TRIPLE}-cpp \
    CXX=/usr/bin/${CROSS_TRIPLE}-g++ \
    LD=/usr/bin/${CROSS_TRIPLE}-ld

# Raspberry Pi is ARMv6+VFP2, Debian armhf is ARMv7+VFP3
# Since this Dockerfile is targeting linux-arm from Raspberry Pi onward,
# we're sticking with it's custom built cross-compiler with hardfp support.
# We could use Debian's armel, but we'd have softfp and loose a good deal
# of performance.
# See: https://wiki.debian.org/RaspberryPi
# We are also using the 4.7 version of the toolchain, so that glibc=2.13

# Instead of cloning the whole repo (>1GB at the of writing this), we want to do a so-called "sparse checkout" with "shallow cloning":
# https://stackoverflow.com/questions/600079/is-there-any-way-to-clone-a-git-repositorys-sub-directory-only/13738951#13738951

RUN mkdir rpi_tools && cd rpi_tools && git init && git remote add -f origin https://github.com/raspberrypi/tools && \
    git config core.sparseCheckout true && echo "arm-bcm2708/gcc-linaro-${CROSS_TRIPLE}-raspbian" >> .git/info/sparse-checkout && \
    git pull --depth=1 origin master && rsync -av arm-bcm2708/gcc-linaro-${CROSS_TRIPLE}-raspbian/ /usr/ && rm -rf ../rpi_tools

# Allow dynamically linked executables to run with qemu-arm
ENV QEMU_LD_PREFIX ${CROSS_ROOT}/libc
ENV QEMU_SET_ENV "LD_LIBRARY_PATH=${CROSS_ROOT}/lib:${CROSS_ROOT}/libc/lib/${CROSS_TRIPLE}/"

ENV DEFAULT_DOCKCROSS_IMAGE dockcross/linux-armv6

COPY Toolchain.cmake ${CROSS_ROOT}/
ENV CMAKE_TOOLCHAIN_FILE ${CROSS_ROOT}/Toolchain.cmake

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
