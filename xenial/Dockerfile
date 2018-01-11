FROM xutl/ubuntu:xenial

LABEL maintainer="xutongle@gmail.com"

# install httpd runtime dependencies
RUN set -xe \
#	&& sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/' /etc/apt/sources.list \
	&& apt-get update \
	&& apt-get install -y --no-install-recommends --no-install-suggests \
		libapr1 \
		libaprutil1 \
		libaprutil1-ldap \
		libapr1-dev \
		libaprutil1-dev \
		liblua5.2 \
		libnghttp2-14 \
		libpcre++0v5 \
		libssl1.0.0\
		libxml2 \
	&& rm -r /var/lib/apt/lists/*

ARG HTTPD_VERSION

ENV HTTPD_VERSION 2.4.29
ENV HTTPD_TGZ_URL http://mirrors.shuosc.org/apache/httpd/httpd-${HTTPD_VERSION}.tar.gz

ENV HTTPD_PREFIX /usr/local/apache2
ENV PATH $HTTPD_PREFIX/bin:$PATH

RUN set -xe \
	&& buildDeps=" \
		bzip2 \
		ca-certificates \
		dpkg-dev \
		gcc \
		liblua5.2-dev \
		libnghttp2-dev \
		libpcre++-dev \
		libssl-dev \
		libxml2-dev \
		zlib1g-dev \
		make \
		curl \
	" \
	\
#	&& sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/' /etc/apt/sources.list \
	&& apt-get update && apt-get install -y --no-install-recommends --no-install-suggests mcrypt $buildDeps && rm -r /var/lib/apt/lists/* \
	\
	&& cd /usr/local/src \
	&& curl -fSL ${HTTPD_TGZ_URL} -o httpd-${HTTPD_VERSION}.tar.gz \
	&& tar zxf httpd-${HTTPD_VERSION}.tar.gz \
	&& rm -rf httpd-${HTTPD_VERSION}.tar.gz \
	&& cd httpd-${HTTPD_VERSION}/ \
	&& gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)" \
	&& ./configure \
		--build="$gnuArch" \
		--prefix="$HTTPD_PREFIX" \
		--enable-mods-shared=reallyall \
	&& make -j "$(nproc)" \
	&& make install \
	&& rm -rf /usr/local/src/httpd-${HTTPD_VERSION} \
	&& sed -ri \
		-e 's!^(\s*CustomLog)\s+\S+!\1 /proc/self/fd/1!g' \
		-e 's!^(\s*ErrorLog)\s+\S+!\1 /proc/self/fd/2!g' \
		"$HTTPD_PREFIX/conf/httpd.conf" \
	&& apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false $buildDeps


COPY httpd-foreground /usr/local/bin/

EXPOSE 80

CMD ["httpd-foreground"]