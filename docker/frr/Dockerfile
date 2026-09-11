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
