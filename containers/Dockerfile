FROM registry.access.redhat.com/ubi9/ubi:latest as builder
RUN dnf install -y make git unzip gcc perl
RUN git clone https://github.com/wg/wrk.git --depth=1
RUN cd wrk && make -j $(nproc)
FROM registry.access.redhat.com/ubi9/ubi:latest
RUN dnf install -y --nodocs https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
COPY centos9-appstream.repo /etc/yum.repos.d/centos9-appstream.repo
RUN dnf update -y && dnf install -y iperf3 \
uperf \
nginx \
iproute \
iputils \
vim-minimal \
procps-ng \
net-tools \
ethtool \
bpftrace \
bcc-tools \
man \
sysstat \
tcpdump \
strace \
bind-utils \
blktrace \
conntrack \
tar \
--enablerepo=centos9-appstream \
&& rm -Rf /var/cache/yum
COPY --from=builder /wrk/wrk /usr/bin/wrk
COPY json.lua json.lua
COPY nginx/root ./
RUN chmod 755 /etc/nginx/ssl
RUN curl -sS -L https://github.com/rsevilla87/hloader/releases/download/v0.2.1/hloader-Linux-v0.2.1-$(arch | sed s/aarch64/arm64/).tar.gz | tar xz -C /usr/bin/

