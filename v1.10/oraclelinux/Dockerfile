# Copyright (C) 2020, Oracle and/or its affiliates.
# Licensed under the Universal Permissive License v 1.0 as shown at https://oss.oracle.com/licenses/upl.

FROM container-registry.oracle.com/os/oraclelinux:7-slim@sha256:fcc6f54bb01fc83319990bf5fa1b79f1dec93cbb87db3c5a8884a5a44148e7bb
LABEL maintainer ="Verrazzano developers <verrazzano_ww@oracle.com>"
LABEL Description="Fluentd docker image for Oracle Linux 7" Vendor="Oracle" Version="1.10.4"
ENV TINI_VERSION=0.18.0
ENV RBENV_ROOT /root/.rbenv
ENV PATH "$RBENV_ROOT/bin:$RBENV_ROOT/shims:$PATH"


# Install ruby-2.6.6
RUN set -eux; \
    yum -y update; \
    yum-config-manager --save --setopt=ol7_ociyum_config.skip_if_unavailable=true ; \
    yum -y install \
        wget \
        which \
        git-core \
        zlib \
        zlib-devel \
        gcc-c++ \
        patch \
        readline \
        readline-devel \
        libyaml-devel \
        libffi-devel \
        openssl-devel \
        make \
        bzip2 \
        autoconf \
        automake \
        libtool \
        bison \
        curl \
        sqlite-devel \
    ; \
    curl -fsSL https://github.com/rbenv/rbenv-installer/raw/HEAD/bin/rbenv-installer | bash ; \
    echo "$PATH" ; \
    rbenv install 2.6.6 ; \
    rbenv global 2.6.6 ; \
    ruby -v ; \
    gem -v ;  \
    bundle -v ; \
    echo 'gem: --no-document' >> /etc/gemrc ; \
    gem install oj -v 3.8.1 ; \
    gem install json -v 2.3.0 ; \
    gem install async-http -v 0.50.7 ; \
    gem install ext_monitor -v 0.1.2 ; \
    gem install fluentd -v 1.10.4 ; \
    dpkgArch="amd64" ; \
    wget -O /usr/local/bin/tini "https://github.com/krallin/tini/releases/download/v$TINI_VERSION/tini-$dpkgArch" ; \
    wget -O /usr/local/bin/tini.asc "https://github.com/krallin/tini/releases/download/v$TINI_VERSION/tini-$dpkgArch.asc" ; \
    export GNUPGHOME="$(mktemp -d)" ; \
    gpg --batch --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 6380DC428747F6C393FEACA59A84159D7001A4E5 ; \
    gpg --batch --verify /usr/local/bin/tini.asc /usr/local/bin/tini ; \
    rm -r /usr/local/bin/tini.asc ; \
    chmod +x /usr/local/bin/tini ; \
    tini -h ; \
    wget -O /tmp/jemalloc-4.5.0.tar.bz2 https://github.com/jemalloc/jemalloc/releases/download/4.5.0/jemalloc-4.5.0.tar.bz2 ; \
    cd /tmp && tar -xjf jemalloc-4.5.0.tar.bz2 && cd jemalloc-4.5.0/ ; \
    ./configure && make ; \
    mv lib/libjemalloc.so.2 /usr/lib ; \
    yum clean all ; \
    rm -rf /var/cache/yum ; \
    rm -rf /tmp/* /var/tmp/* /usr/lib/ruby/gems/*/cache/*.gem ; \
    groupadd -r fluent && useradd -r -g fluent fluent ; \
    # for log storage (maybe shared with host)
    mkdir -p /fluentd/log ;\
    # configuration/plugins path (default: copied from .)
    mkdir -p /fluentd/etc /fluentd/plugins ; \
    chown -R fluent /fluentd && chgrp -R fluent /fluentd

COPY v1.10/oraclelinux/fluent.conf /fluentd/etc/
COPY v1.10/oraclelinux/entrypoint.sh /bin/

#Copy license and README to image
COPY LICENSE README.md THIRD_PARTY_LICENSES.txt /license/

ENV FLUENTD_CONF="fluent.conf"

ENV LD_PRELOAD="/usr/lib/libjemalloc.so.2"
EXPOSE 24224 5140

USER fluent
ENTRYPOINT ["tini",  "--", "/bin/entrypoint.sh"]
CMD ["fluentd"]
