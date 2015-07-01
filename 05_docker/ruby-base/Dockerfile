FROM ubuntu:14.04

RUN apt-get update -q && apt-get install -qy \
      build-essential \
      libpq-dev \
      zlib1g-dev \
      libssl-dev \
      libreadline6-dev \
      libyaml-dev \
      libsqlite3-dev \
      nginx \
      supervisor \
      ca-certificates

# If you want to make SSL connections to servers using certificates signed by
# your private CA, copy your cacert.pem file into config/ and uncomment!
# COPY config/cacert.pem /usr/local/share/ca-certificates/local.devel.crt
# RUN /usr/sbin/update-ca-certificates

COPY config/nginx.conf /etc/nginx/nginx.conf

ADD http://cache.ruby-lang.org/pub/ruby/2.2/ruby-2.2.2.tar.gz /tmp/

WORKDIR /tmp
RUN tar -xzf ruby-2.2.2.tar.gz && \
      cd /tmp/ruby-2.2.2 && \
      ./configure --disable-install-doc && make && make install && \
      cd /tmp && rm -rf ruby-2.2.2/ && rm -f ruby-2.2.2.tar.gz

RUN gem update --system --no-document && \
      gem install bundler --no-document

RUN adduser --disabled-password --home=/app --gecos "" app
