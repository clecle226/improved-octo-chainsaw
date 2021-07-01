FROM debian:stable-slim

RUN apt-get update \
    && apt-get install -y build-essential libssl-dev libio-socket-ssl-perl liblwp-protocol-https-perl zlib1g-dev \
    && apt-get install -y libpq-dev \
    && apt-get install -y git \
    && cpan Carton \
    && mkdir /var/www && cd /var/www \
    && git clone https://framagit.org/fiat-tux/hat-softwares/lufi.git /var/www/ \
    && carton install --deployment --without=test --without=sqlite --without=mysql --without=ldap --without=ldap --without=swift-storage

WORKDIR /var/www/

EXPOSE 8081

CMD ["carton", "exec", "hypnotoad", "script/lufi"]
