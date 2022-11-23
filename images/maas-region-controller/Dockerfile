ARG FROM=phusion/baseimage:focal-1.2.0
FROM ${FROM}

ARG HTTP_PROXY
ARG HTTPS_PROXY
ARG NO_PROXY
ARG http_proxy
ARG https_proxy
ARG no_proxy

ENV DEBIAN_FRONTEND noninteractive
ENV container docker

RUN apt-get -qq update \
 && apt-get install -y \
        avahi-daemon \
        jq \
        patch \
        software-properties-common \
        sudo \
        ca-certificates \
# Don't start any optional services except for the few we need.
# (specifically, don't start avahi-daemon)
 && find /etc/systemd/system \
         /lib/systemd/system \
         -path '*.wants/*' \
         -not -name '*journald*' \
         -not -name '*systemd-tmpfiles*' \
         -not -name '*systemd-user-sessions*' \
         -exec rm \{} \; \
# Install maas from the ppa
 && add-apt-repository -yu ppa:maas/3.2 \
 && apt-get update \
 && apt-get install -y \
        maas-region-api \
        # tcpdump is required by /usr/lib/maas/beacon-monitor
        tcpdump \
 && rm -rf /var/lib/apt/lists/*

# echo journalctl logs to the container's stdout
COPY journalctl-to-tty.service /etc/systemd/system/journalctl-to-tty.service
RUN systemctl enable journalctl-to-tty.service

COPY 3.2.6_add_regiond_tgt_conf.patch /tmp/3.2.6_add_regiond_tgt_conf.patch
RUN cd /usr/lib/python3/dist-packages/maasserver && patch -p3 < /tmp/3.2.6_add_regiond_tgt_conf.patch

# quiet sudo for the maas user
RUN umask 0337; echo 'Defaults:maas !pam_session, !syslog' > /etc/sudoers.d/99-maas-no-log

# maas reeeeeeally likes systemd
CMD ["/bin/bash", "-c", "exec /sbin/init --log-target=console 3>&1"]
