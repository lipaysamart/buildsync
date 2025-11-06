FROM registry.k8s.io/ingress-nginx/controller:v1.8.1

USER root

RUN apk add --no-cache make gcc unzip wget

# install luaroncks
RUN wget https://luarocks.org/releases/luarocks-3.11.0.tar.gz && \
    tar zxpf luarocks-3.11.0.tar.gz && \
    cd luarocks-3.11.0 && \
    ./configure && \
    make && \
    make install && \
    cd .. && \
    rm -rf luarocks-3.11.0 luarocks-3.11.0.tar.gz

RUN luarocks install ingress-nginx-safeline && \
    ln -s /usr/local/share/lua/5.1/safeline /etc/nginx/lua/plugins/safeline

USER www-data