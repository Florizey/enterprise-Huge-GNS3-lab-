# FRR router image for the GNS3 "FRRDocker" / "FRRDockerLite" appliances.
# Build: docker build -t gns3-frr:latest ~/enterprise/docker/frr
FROM gns3/frr:latest

# Ubuntu hosts ship AppArmor profiles for /usr/lib/frr/{bgpd,ospfd,ospf6d,staticd,...}
# (apparmor package, /etc/apparmor.d/<daemon>). They attach by path even inside the
# unconfined GNS3 container and deny libc mprotect, so those daemons die at startup:
# static routes never reach the kernel and 'write memory' fails with
# "Not all daemons are up". Move the daemons to a path those profiles don't match;
# the symlink keeps every /usr/lib/frr/... reference (start_command, scripts) working.
RUN mv /usr/lib/frr /usr/libexec/frr \
    && ln -s /usr/libexec/frr /usr/lib/frr

# /etc/pam.d/frr has no account stack, so every vtysh call printed
# "vtysh_pam: Failed in account validation".
RUN printf 'account\tsufficient\tpam_permit.so\n' >> /etc/pam.d/frr

# iptables for NAT/PAT on edge routers (e.g. MASQUERADE out the NAT-facing interface).
# Rules are applied at boot from the node's /etc/network/interfaces, which GNS3 persists.
# isc-dhcp-relay (dhcrelay) to relay client DHCP to the DHCP server; FRR has no relay of
# its own. Installed unconfigured: set SERVERS/INTERFACES/OPTIONS in /etc/default/isc-dhcp-relay.
RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
       iptables isc-dhcp-relay \
    && rm -rf /var/lib/apt/lists/*

# Start the DHCP relay on every boot: docker-start becomes a wrapper that launches
# dhcp-relay-boot, then runs FRR's original script. The relay config is kept in
# /etc/frr (persisted by GNS3) and /etc/default/isc-dhcp-relay points there; the
# pristine default is kept as the template dhcp-relay-boot seeds new nodes from.
COPY --chmod=755 dhcp-relay-boot /usr/local/sbin/dhcp-relay-boot
COPY --chmod=755 docker-start-wrapper /usr/libexec/frr/docker-start.wrapper
RUN mkdir -p /usr/local/share/dhcp-relay \
    && mv /etc/default/isc-dhcp-relay /usr/local/share/dhcp-relay/isc-dhcp-relay.default \
    && ln -s /etc/frr/isc-dhcp-relay /etc/default/isc-dhcp-relay \
    && mv /usr/libexec/frr/docker-start /usr/libexec/frr/docker-start.frr \
    && mv /usr/libexec/frr/docker-start.wrapper /usr/libexec/frr/docker-start
