FROM debian:stable-slim as builder

#Preparer le terrain
RUN printf "deb-src http://deb.debian.org/debian/ stable main \n\
deb-src http://deb.debian.org/debian/ stable-updates main \n\
deb-src http://security.debian.org/debian-security stable-security main \
    " >> /etc/apt/sources.list.d/nginx-src.list \
    && apt-get update \
    && apt-get build-dep -y nginx \
    && mkdir /usr/src/nginx \
    && cd /usr/src/nginx && apt-get source nginx \
    && apt-get install -y geoip-database libgeoip1
    
#PREP CROSSARCH: armhf (OPI)
RUN sudo dpkg --add-architecture armhf \
    && sudo apt-get update \
    && sudo apt-get -y install build-essential crossbuild-essential-armhf \
    && sudo apt-get build-dep -y -aarmhf <package>
    
#BROTLI
RUN apt-get install -y libbrotli-dev brotli git \
    && git clone https://github.com/google/ngx_brotli /tmp/ngx_brotli
#CHANGEMENT NOM SERVEUR
RUN VERSION_NGINX=`apt-cache show nginx-extras/stable | grep Version | awk '{split($0,a);print a[2]}' | awk '{split($0,a,"-"); print a[1]}'` \
    && sed -i -e 's#Server: nginx#Server: CMaBite!!!#' "/usr/src/nginx/nginx-$VERSION_NGINX/src/http/ngx_http_header_filter_module.c" \
    && sed -i -e 's#"Server: " NGINX_VER_BUILD#"Server: CMaBite!!!"#' "/usr/src/nginx/nginx-$VERSION_NGINX/src/http/ngx_http_header_filter_module.c" \
    && sed -i -e 's#"Server: " NGINX_VER#"Server: CMaBite!!!"#' "/usr/src/nginx/nginx-$VERSION_NGINX/src/http/ngx_http_header_filter_module.c"

#QUIC(HE) de cloudflare
#RUN apt-get build-dep -y openssl \
#    && apt-get install -y cmake curl \
#    && curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y \
#    && export PATH="$HOME/.cargo/bin:$PATH" \
#    && VERSION_NGINX=`apt-cache show nginx-extras/stable | grep Version | awk '{split($0,a);print a[2]}' | awk '{split($0,a,"-"); print a[1]}'` \
#    && git clone --recursive https://github.com/cloudflare/quiche /tmp/ngx_quic \
#    && apt-get install -y mercurial libpcre3 libpcre3-dev gcc make autoconf zlib1g zlib1g-dev \
#    && cd /usr/src/nginx/nginx-$VERSION_NGINX/ \
#    && patch -p01 < /tmp/ngx_quic/extras/nginx/nginx-1.16.patch

#&& sed -i -e 's#^extras_configure_flags.*#extras_configure_flags := --add-module=/tmp/ngx_brotli --with-http_v3_module --with-openssl=/tmp/ngx_quic/deps/boringssl --with-quiche=/tmp/ngx_quic \\#' "/usr/src/nginx/nginx-$VERSION_NGINX/debian/rules" \

#COMPILATION
RUN VERSION_NGINX=`apt-cache show nginx-extras/stable | grep Version | awk '{split($0,a);print a[2]}' | awk '{split($0,a,"-"); print a[1]}'` \
    && sed -i -e 's#^extras_configure_flags.*#extras_configure_flags := --add-module=/tmp/ngx_brotli \\#' "/usr/src/nginx/nginx-$VERSION_NGINX/debian/rules" \
    && cd /usr/src/nginx/nginx-$VERSION_NGINX/ \
    && dpkg-buildpackage -b \
    && CONFIG_SITE=/etc/dpkg-cross/cross-config.<host_arch>  DEB_BUILD_OPTIONS=nocheck dpkg-buildpackage -aarmhf -Pcross,nocheck


# -> Copy sur un debian vierge et install
# /bin/sh -c 'dpkg -i nginx-common_*.deb libnginx-mod*.deb nginx-extras*.deb'

FROM scratch

#MAINTAINER clecle226 "clecle226@noknow.ovh"

COPY --from=builder /usr/src/nginx/*.deb /

#Extraire .deb
#docker save nginx:deb | tar -x --wildcards */layer.tar --to-stdout | tar -x -C ./DEB_NGINX/
#docker buildx build -f ./Docker-Nginx/Dockerfile -o - - | tar -x --wildcards */layer.tar --to-stdout | tar -x -C ./DEB_NGINX/
#docker  build ./Docker-Nginx/ - | tar -x --wildcards */layer.tar --to-stdout | tar -x -C ./DEB_NGINX/
#docker build -f ./Docker-Nginx/Dockerfile - | tar -x --wildcards */layer.tar --to-stdout | tar -x -C ./DEB_NGINX/
#docker buildx build -o - ./Docker-Nginx/.  | tar -x --wildcards */layer.tar --to-stdout | tar -x -C ./DEB_NGINX/

#docker buildx build --no-cache -o - ./Docker-Nginx/. | tar -x -C ./DEB_NGINX/
