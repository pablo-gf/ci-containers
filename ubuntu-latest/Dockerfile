FROM ubuntu:latest as base
LABEL version="2"

# actionlint - for GitHub workflow file validation
# (version pinned to commit hash of v1.7.1)
FROM golang:1.23 as build
RUN mkdir /app
WORKDIR /app
ENV CGO_ENABLED 0
RUN git clone https://github.com/rhysd/actionlint.git
WORKDIR /app/actionlint
RUN git reset --hard 03d0035 # v1.7.7
RUN go build -o /usr/local/bin/actionlint ./cmd/actionlint
# copy built binary from build stage to final image
FROM ubuntu:latest
WORKDIR /root
COPY --from=build /usr/local/bin/actionlint /usr/local/bin/actionlint

ARG ARCH
ENV ARCH=${ARCH}

RUN DEBIAN_FRONTEND=noninteractive TZ=Europe/Zurich && \
    ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && \
    echo $TZ > /etc/timezone && \
    apt-get update -qq && \
    apt-get dist-upgrade -y && \
    apt-get install -y tzdata && \
    dpkg-reconfigure --frontend noninteractive tzdata && \
    apt-get install -y apt-utils wget \
                       autoconf \
                       automake \
                       astyle \
                       clang \
                       clang-format \
                       clang-tools \
                       cmake \
                       curl \
                       default-jdk-headless \
                       docker.io \
                       gcc \
                       gcc-mingw-w64 \
                       g++ \
                       git \
                       libgmp-dev \
                       libtool \
                       libssl-dev \
                       libunwind-dev \
                       make \
                       ninja-build \
                       npm \
                       opam \
                       pkg-config \
                       python3 \
                       python3-git \
                       python3-pytest \
                       python3-pytest-xdist \
                       python3-psutil \
                       python3-pip \
                       python3-tabulate \
                       python3-jinja2 \
                       python3-requests \
                       zlib1g-dev \
                       maven \
                       valgrind \
                       flex bison \
                       lsb-release software-properties-common \
                       lcov \
                       graphviz \
                       llvm \
                       golang \
                       doxygen \
                       yamllint

# set up jasmin compiler for libjade
RUN opam init --yes --auto-setup && opam install --confirm-level=unsafe-yes --destdir=/usr/local jasmin.2023.06.3

# install ajv for CBOM validation
RUN npm -g install ajv ajv-cli

# set up Kyberslash patch for Valgrind-based constant-time testing
COPY valgrind-try-patch-20250805.txt /tmp/
COPY valgrind-varlat-patch-20250805.txt /tmp/
COPY valgrind-varlat-sup-block.txt /tmp/

RUN git clone git://sourceware.org/git/valgrind.git /tmp/valgrind_varlat_src && \
    cd /tmp/valgrind_varlat_src && \
    git checkout 112f1080b7c21e37dfce0a2e589d0dc7aa115afa && \
    git apply /tmp/valgrind-try-patch-20250805.txt && \
    git apply /tmp/valgrind-varlat-patch-20250805.txt && \
    git apply /tmp/valgrind-varlat-sup-block.txt && \
    ./autogen.sh && \
    ./configure --prefix=/opt/valgrind_varlat && \
    make -j"$(nproc)" && \
    make install && \
    mv /opt/valgrind_varlat/bin/valgrind /opt/valgrind_varlat/bin/valgrind_varlat && \
    cd / && rm -rf /tmp/valgrind_varlat_src /tmp/*.txt

ENV PATH="/opt/valgrind_varlat/bin:${PATH}"

# set JAVA_HOME for liboqs-java builds
ENV JAVA_HOME="/usr/lib/jvm/default-java"

# Activate if we want to test specific OpenSSL3 versions:
# RUN cd /root && git clone --depth 1 --branch openssl-3.0.7 https://github.com/openssl/openssl.git && cd openssl && LDFLAGS="-Wl,-rpath -Wl,/usr/local/openssl3/lib64"  ./config --prefix=/usr/local/openssl3 && make -j && make install