FROM debian:buster-slim

# ensure local openssl is preferred over distribution openssl
ENV PATH /usr/local/bin:$PATH

ENV OPENSSL_VERSION 1.1.0k
ENV GPG_KEY 8657ABB260F056B1E5190839D9C4D26D0E604491 5B2545DAB21995F4088CEFAA36CEE4DEB00CFE33 ED230BEC4D4F2518B9D7DF41F0DB4D21C1D35231 C1F33DD8CE1D4CC613AF14DA9195C48241FBF7DD 	7953AC1FBC3DC8B3B292393ED5E9E43F7DF9EE8C E5E52560DD91C556DDBDA5D02064C53641C25E5D

RUN set -x \
    && buildDeps=' \
        ca-certificates \
        gcc \
        libc6-dev \
        make \
        perl \
        wget \
    ' \
    && apt-get update && apt-get install -y --no-install-recommends $buildDeps \
    && rm -rf /var/lib/apt/lists/* \
    \
    && wget -O openssl.tar.gz "https://www.openssl.org/source/openssl-$OPENSSL_VERSION.tar.gz" \
    && wget -O openssl.tar.gz.asc "https://www.openssl.org/source/openssl-$OPENSSL_VERSION.tar.gz.asc" \
    && export GNUPGHOME="$(mktemp -d)" \
    && gpg --keyserver ha.pool.sks-keyservers.net --recv-keys $GPG_KEY \
    && gpg --batch --verify openssl.tar.gz.asc openssl.tar.gz \
    && rm -r "$GNUPGHOME" openssl.tar.gz.asc \
    && mkdir -p /usr/src/openssl \
    && tar -xzC /usr/src/openssl --strip-components=1 -f openssl.tar.gz \
    && rm openssl.tar.gz \
# `make test` needs to be run as a non-root user so we create user & group 'openssl' to build with
    && groupadd -r openssl && useradd -r -m -g openssl openssl \
    && chown -R openssl:openssl /usr/src/openssl \
    \
    && cd /usr/src/openssl \
    && su openssl -c ' \
        perl ./Configure linux-x86_64 \
            enable-ec_nistp_64_gcc_128 \
        && make \
        && make test \
    ' \
# install openssl as root
    && cd /usr/src/openssl \
    && make install_sw install_ssldirs \
    && ldconfig \
    \
# bootstrap with the system CA certificates, a la Homebrew
    && cp /etc/ssl/certs/*.pem /usr/local/ssl/certs/ \
    && c_rehash /usr/local/ssl/certs \
    \
    && apt-get purge -y --auto-remove $buildDeps \
    && rm -rf /usr/src/openssl \
    && userdel -r openssl

CMD ["openssl"]
