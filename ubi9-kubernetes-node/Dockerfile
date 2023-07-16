FROM registry.access.redhat.com/ubi9/ubi-init:9.2-5 
LABEL maintainer="Red Hat, Inc."

LABEL com.redhat.component="ubi9-init-container"
LABEL com.redhat.license_terms="https://www.redhat.com/en/about/red-hat-end-user-license-agreements#UBI"
LABEL name="ubi9/ubi9-init"
LABEL version="9.2"

LABEL summary="Provides the latest release of the Red Hat Universal Base Image 9 Init for multi-service containers."
LABEL description="The Universal Base Image Init is designed is designed to run an init system as PID 1 for running multi-services inside a container. This base image is freely redistributable, but Red Hat only supports Red Hat technologies through subscriptions for Red Hat products. This image is maintained by Red Hat and updated regularly."
LABEL io.k8s.display-name="Red Hat Universal Base Image 9 Init"
LABEL io.openshift.expose-services=""
LABEL usage="Do not use directly. Use as a base image for daemons. Install chosen packages and 'systemctl enable' them."

CMD ["/sbin/init"]

STOPSIGNAL SIGRTMIN+3

#mask systemd-machine-id-commit.service - partial fix for https://bugzilla.redhat.com/show_bug.cgi?id=1472439
RUN systemctl mask systemd-remount-fs.service dev-hugepages.mount sys-fs-fuse-connections.mount systemd-logind.service getty.target console-getty.service systemd-udev-trigger.service systemd-udevd.service systemd-random-seed.service systemd-machine-id-commit.service

#RUN mkdir /mnt

WORKDIR /mnt

#TODO - TEST OUT GLOBING AND WILCARDS VS EXTERNAL LISTS AND VARIABLE EXPANSION
#RUN find /mnt/BaseOS -name "*podman*x86_64.rpm" -printf '%f\n' -exec dnf install -y '{}' \+;
#RUN ["/bin/bash", "-c", "find /mnt/BaseOS -name \"*podman*x86_64.rpm\" -printf '%f\n' -exec echo '{}' \\;"]
#RUN ls /bin/s*
#RUN [/bin/sh, "-c", "find /mnt/BaseOS -name "*podman*x86_64.rpm" -printf '%f\n' -exec dnf install '{}' \;"]


RUN dnf -y install podman \
&& dnf -y install podman-catatonit \
&& dnf -y install cockpit-podman \
&& dnf -y install /mnt/BaseOS/Packages/kernel-core-5.14.0-162.6.1.el9_1.x86_64.rpm \
&& dnf -y install /mnt/BaseOS/Packages/net-tools-2.0-0.62.20160912git.el9.x86_64.rpm \
&& dnf -y install /mnt/BaseOS/Packages/iputils-20210202-7.el9.x86_64.rpm \
&& dnf -y install /mnt/AppStream/Packages/runc-1.1.4-1.el9.x86_64.rpm \
&& dnf -y install /mnt/BaseOS/Packages/kernel-modules-5.14.0-162.6.1.el9_1.x86_64.rpm \
&& dnf -y install /mnt/BaseOS/Packages/kmod-28-7.el9.x86_64.rpm \
&& dnf -y install /mnt/BaseOS/Packages/kmod-kvdo-8.2.0.21-47.el9_1.x86_64.rpm \
&& dnf -y install /mnt/BaseOS/Packages/kmod-libs-28-7.el9.x86_64.rpm \
&& dnf -y install iptables-libs \
&& dnf -y install iptables-nft \
#&& dnf -y install /mnt/BaseOS/Packages/iptables-libs-1.8.8-4.el9.x86_64.rpm \
#&& dnf -y install /mnt/BaseOS/Packages/iptables-nft-1.8.8-4.el9.x86_64.rpm \
&& dnf -y install /mnt/AppStream/Packages/socat-1.7.4.1-5.el9.x86_64.rpm \
&& dnf -y install /mnt/BaseOS/Packages/ethtool-5.16-1.el9.x86_64.rpm \
&& dnf -y install /mnt/BaseOS/Packages/libbpf-0.6.0-1.el9.x86_64.rpm \
&& dnf -y install /mnt/BaseOS/Packages/procps-ng-3.3.17-8.el9.x86_64.rpm \
&& dnf -y install /mnt/BaseOS/Packages/openssh-8.7p1-24.el9_1.x86_64.rpm \
&& dnf -y install /mnt/AppStream/Packages/libbpf-tools-0.24.0-4.el9.x86_64.rpm \
&& dnf -y install /mnt/BaseOS/Packages/iproute-5.18.0-1.el9.x86_64.rpm \
&& dnf -y install /mnt/BaseOS/Packages/iproute-tc-5.18.0-1.el9.x86_64.rpm \
&& dnf -y install /mnt/AppStream/Packages/libnetfilter_cthelper-1.0.0-22.el9.x86_64.rpm \
&& dnf -y install /mnt/BaseOS/Packages/openssh-server-8.7p1-24.el9_1.x86_64.rpm \
&& dnf -y install /mnt/AppStream/Packages/libnetfilter_cttimeout-1.0.0-19.el9.x86_64.rpm \
&& dnf -y install /mnt/AppStream/Packages/libnetfilter_queue-1.0.5-1.el9.x86_64.rpm \
&& dnf -y install /mnt/AppStream/Packages/conntrack-tools-1.4.5-15.el9.x86_64.rpm \
&& dnf -y install gettext \
&& dnf clean all

RUN mkdir -p /opt/cni/bin
WORKDIR /opt/cni/bin
ADD software/cni-plugins-linux-amd64-v1.3.0.tgz .
COPY config/env.sh /etc/profile.d
COPY config/10-containerd-net.conflist /etc/cni/net.d/10-containerd-net.conflist
RUN mkdir /etc/containerd
COPY config/config.toml /etc/containerd

WORKDIR /usr/local
ADD software/containerd-1.7.1-linux-amd64.tar.gz .
COPY config/containerd.service /usr/lib/systemd/system/containerd.service

WORKDIR /usr/local/bin
ADD software/crictl-1.27.0-linux-amd64.tar.gz .

COPY software/kubelet /usr/local/bin
COPY config/kubelet.service /etc/systemd/system
RUN mkdir -p /var/lib/kubelet
COPY config/config.yaml /var/lib/kubelet
RUN mkdir -p /etc/kubernetes/manifests
RUN mkdir /root/.kube
COPY config/admin.conf /etc/kubernetes/
COPY config/admin.conf /root/.kube/config
#COPY config/kube-apiserver.yaml /etc/kubernetes/manifests

COPY config/user1* /root

COPY software/kubeadm /usr/local/bin
COPY config/10-kubeadm.conf /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

WORKDIR /root
COPY software/kubectl .
RUN install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

COPY config/k8s.conf /etc/modules-load.d
COPY config/etc-sysctl.d-k8s.conf /etc/sysctl.d/k8s.conf
#RUN sysctl --system

#RUN mkdir -p /lib/modules/5.14.0-162.6.1.el9_1.x86_64
#COPY config/modules.order /lib/modules/5.14.0-162.6.1.el9_1.x86_64

#RUN modprobe overlay && modprobe br_netfilter
#RUN lsmod | grep br_netfilter && lsmod | grep overlay

ENV NAME user1
ENV PASS user1
RUN useradd -d /home/user1 -g root -m -p $(echo "$PASS" | openssl passwd -1 -stdin) $NAME

RUN swapoff -a
RUN systemctl enable kubelet && systemctl enable containerd && systemctl enable sshd 
