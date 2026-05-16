FROM public.ecr.aws/ubuntu/ubuntu:22.04

COPY run_duo.sh /app/

RUN set -xe && \
    apt-get update && \
    apt-get install -y \
        gcc \
        make \
        zlib1g-dev \
        libffi-dev \
        tar \
        wget \
        gzip \
        perl \
        jq \
        procps \
        diffutils \
        python3 \
        python3-dev \
        libssl-dev \
        libsqlite3-dev \
        libbz2-dev \
        liblzma-dev && \
    wget https://dl.duosecurity.com/duoauthproxy-latest-src.tgz && \
    tar xzf duoauthproxy-*.tgz && \
    rm duoauthproxy-*.tgz && \
    cd duoauthproxy-*-src && \
    make && \
    cd /duo*/duo*/ && \
    ./install --install-dir=/opt/duoauthproxy --service-user=nobody --create-init-script=yes --log-group=duo_authproxy_grp && \
    rm -fr /duo* && \
    apt-get clean && \
    chmod +x /app/run_duo.sh && \
    wget -O /tmp/pyopenssl.whl "https://files.pythonhosted.org/packages/fb/7d/d4f7d908fa8415571771b30669251d57c3cf313b36a856e6d7548ae01619/pyopenssl-26.0.0-py3-none-any.whl" && \
    /opt/duoauthproxy/usr/local/bin/pip3 install --no-index --find-links=/tmp /tmp/pyopenssl.whl && \
    rm /tmp/pyopenssl.whl

EXPOSE 1812/udp
ENTRYPOINT ["/app/run_duo.sh"]
