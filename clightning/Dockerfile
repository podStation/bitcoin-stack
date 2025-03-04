FROM alpine:3.10 as clightning

RUN sed -i 's/http\:\/\/dl-cdn.alpinelinux.org/https\:\/\/alpine.global.ssl.fastly.net/g' /etc/apk/repositories
RUN apk --no-cache add autoconf
RUN apk --no-cache add automake
RUN apk --no-cache add boost-dev
RUN apk --no-cache add build-base
RUN apk --no-cache add chrpath
RUN apk --no-cache add file
RUN apk --no-cache add gnupg
RUN apk --no-cache add libevent-dev
RUN apk --no-cache add libressl
RUN apk --no-cache add libressl-dev
RUN apk --no-cache add libtool
RUN apk --no-cache add linux-headers
RUN apk --no-cache add protobuf-dev
RUN apk --no-cache add zeromq-dev
RUN apk --no-cache add py3-mako
RUN apk --no-cache add git
RUN apk --no-cache add gmp-dev
RUN apk --no-cache add sqlite-dev
RUN apk --no-cache add gettext
RUN set -ex \
  && for key in \
    BFF0F67810C1EED1 \
  ; do \
    gpg --batch --keyserver keyserver.ubuntu.com --recv-keys "$key" || \
    gpg --batch --keyserver pgp.mit.edu --recv-keys "$key" || \
    gpg --batch --keyserver keyserver.pgp.com --recv-keys "$key" || \
    gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys "$key" || \
    gpg --batch --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys "$key" ; \
  done

ENV CLIGHTNING_VERSION=0.9.1
ENV CLIGHTNING_PREFIX=/opt/clightning

RUN wget https://github.com/ElementsProject/lightning/releases/download/v${CLIGHTNING_VERSION}/SHA256SUMS
RUN wget https://github.com/ElementsProject/lightning/releases/download/v${CLIGHTNING_VERSION}/SHA256SUMS.asc
RUN wget https://github.com/ElementsProject/lightning/releases/download/v${CLIGHTNING_VERSION}/clightning-v${CLIGHTNING_VERSION}.zip
# @todo get the correct source bundle checksum from somewhere
# RUN gpg --verify SHA256SUMS.asc
# RUN grep " release/clightning-v${CLIGHTNING_VERSION}.zip\$" SHA256SUMS | sed 's/ release\///g' | sha256sum -c -
RUN unzip -q clightning-v${CLIGHTNING_VERSION}.zip

WORKDIR /clightning-v${CLIGHTNING_VERSION}

RUN ./configure --prefix=${CLIGHTNING_PREFIX}
RUN make -j$(nproc)
RUN make install
RUN strip ${CLIGHTNING_PREFIX}/bin/lightning-cli
RUN strip ${CLIGHTNING_PREFIX}/bin/lightningd

# Build stage for compiled artifacts
FROM alpine:3.10

RUN adduser -S clightning
RUN sed -i 's/http\:\/\/dl-cdn.alpinelinux.org/https\:\/\/alpine.global.ssl.fastly.net/g' /etc/apk/repositories
RUN apk --no-cache add \
  bash \
  gmp \
  boost \
  boost-program_options \
  libevent \
  libressl \
  libzmq \
  su-exec \
  # enable support for custom nodejs plugins
  nodejs \
  npm

ENV CLIGHTNING_DATA=/home/clightning/.lightning
ENV CLIGHTNING_PREFIX=/opt/clightning
ENV PATH=${CLIGHTNING_PREFIX}/bin:$PATH

COPY --from=clightning /opt /opt
COPY docker-entrypoint.sh /entrypoint.sh

VOLUME ["/home/clightning/.lightning"]

ADD ./lightning.conf /home/clightning/.lightning/lightning.conf

ENTRYPOINT ["/entrypoint.sh"]

CMD ["lightningd"]