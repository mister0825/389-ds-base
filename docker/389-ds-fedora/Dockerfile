# --- BEGIN COPYRIGHT BLOCK ---
# Copyright (C) 2017 Red Hat, Inc.
# All rights reserved.
#
# License: GPL (version 3 or any later version).
# See LICENSE for details.
# --- END COPYRIGHT BLOCK ---

FROM fedora:latest
MAINTAINER 389-devel@lists.fedoraproject.org

EXPOSE 3389 3636

# install dependencies
RUN dnf upgrade -y \
  && dnf groupinstall -y 'C Development Tools and Libraries' \
  && dnf install --setopt=strict=False -y @buildsys-build rpm-build npm bzip2 git rsync \
  bzip2-devel cracklib-devel cyrus-sasl-devel doxygen icu krb5-devel libcmocka-devel libdb-devel \
  libevent-devel libicu-devel net-snmp-devel nspr-devel nss-devel openldap-devel \
  openssl-devel pam-devel pcre-devel procps-ng python3-argcomplete python3-argparse-manpage \
  python3-devel python3-ldap python3-libselinux python3-policycoreutils python3-pyasn1 \
  python3-pyasn1-modules systemd-devel zlib-devel \
  `grep -E "^(Build)?Requires" rpm/389-ds-base.spec.in \
  | grep -v -E '(name|MODULE)' \
  | awk '{ print $2 }' \
  | sed 's/%{python3_pkgversion}/3/g' \
  | grep -v "^/" \
  | grep -v pkgversion \
  | sort | uniq \
  | tr '\n' ' '` \
  && dnf clean all

# clone 389-ds-base repo 
WORKDIR /usr/local/src/
RUN git clone https://pagure.io/389-ds-base.git

# build
WORKDIR /usr/local/src/389-ds-base/
RUN SKIP_AUDIT_CI=1 make -f rpm.mk rpms || sh -c 'echo "build failed, sleeping for some time to allow you debug" ; sleep 3600'

RUN dnf install -y dist/rpms/*389*.rpm && \
    dnf clean all
    
# Link some known static locations to point to /data
RUN mkdir -p /data/config && \
  mkdir -p /data/ssca && \
  mkdir -p /data/run && \
  mkdir -p /var/run/dirsrv && \
  ln -s /data/config /etc/dirsrv/slapd-localhost && \
  ln -s /data/ssca /etc/dirsrv/ssca && \
  ln -s /data/run /var/run/dirsrv && \
  chown -R dirsrv.dirsrv /data && \
  chown -R dirsrv.dirsrv /var/run/dirsrv && \
  chgrp -R dirsrv /etc/dirsrv 
  
VOLUME /data

USER dirsrv

HEALTHCHECK --start-period=5m --timeout=5s --interval=5s --retries=2 \
  CMD /usr/libexec/dirsrv/dscontainer -H

CMD [ "/usr/libexec/dirsrv/dscontainer", "-r" ]
