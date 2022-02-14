FROM weaveworks/ignite-ubuntu:20.04 as builder

RUN apt-get update && apt-get install -y \
    make gcc zip g++ libsctp-dev lksctp-tools \
    python3-pip && \
    pip3 install cmake==3.22.2

WORKDIR /opt

RUN wget -O UERANSIM.zip https://github.com/aligungr/UERANSIM/archive/e1afcdaf639b7148d27c23e04b43cbbfdda48133.zip && \
    unzip UERANSIM.zip && \
    cd UERANSIM-* && \
    rm -fr logs && \
    mkdir -p /opt/out && \
    cmake -DCMAKE_BUILD_TYPE=Release -G "CodeBlocks - Unix Makefiles" . -B cmake-build-release && \
    cmake --build cmake-build-release --target nr-ue -j && \
    cp cmake-build-release/nr-ue /opt/out/

FROM weaveworks/ignite-ubuntu:20.04

RUN apt-get update && apt-get install -y libsctp-dev lksctp-tools iproute2

COPY --from=builder --chown=root:root /opt/out/nr-ue /usr/bin