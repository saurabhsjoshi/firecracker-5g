FROM weaveworks/ignite-ubuntu:20.04

# Install Open5gs and disable all services by default
RUN apt-get update && \
    curl -fsSL https://deb.nodesource.com/setup_14.x | sudo -E bash - && \
    apt-get install -y software-properties-common nodejs iptables && \
    add-apt-repository -y ppa:open5gs/latest && \
    apt-get update && \
    apt-get install -y open5gs && \
    systemctl disable open5gs-mmed open5gs-sgwcd open5gs-smfd open5gs-amfd \
    open5gs-sgwud open5gs-upfd open5gs-hssd open5gs-pcrfd open5gs-nrfd \
    open5gs-ausfd open5gs-udmd open5gs-pcfd open5gs-nssfd open5gs-bsfd \
    open5gs-udrd